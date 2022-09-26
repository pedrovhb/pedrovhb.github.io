---
title: "Neat mypy trick - catch runtime exceptions with `typing.Protocol` and class variables"
date: 2022-09-26T11:23:00-03:00
draft: false
------------

I'm writing a little utility library that among other things deals with asynchronously managing subprocesses.
One of the things I want it to do is to make it easier to process output for arbitrary command line utilities.
To do that, you'll subclass the provided base class and implement the relevant methods.


Some programs will output lines of text that are useful as streams, and others really only make sense to look at when the process is finished.
To make it easier to deal with both cases, I want to provide a way to get the output as a stream, or as a single string when the process is finished.
Some programs have a lot of output though, and it's not always desirable to keep it all in memory.
Take for instance encoding a video through ffmpeg using an stdout pipe as the output.
It may not even be possible at all to fit everything in memory.


It'd be nice, then, for the base class to provide the choice of whether to keep the entire stdout (or stderr) output in memory or not.
One way to do this is to set a class variable in the subclass, and then check that in the base class.
This avoids having different classes for each case, and makes it easy to add the functionality to existing subclasses.
It also provides the user with a footgun, though - if they don't set the class variable flag to "keep the entire buffer", they'll get an exception at runtime when trying to access it.
It's not a huge deal, but it'd be nice to avoid it.
Enter static typing.


We can use mypy's `Protocol` to define a protocol that has a class variable, and then use that protocol as the self argument type on [method overloads](https://mypy.readthedocs.io/en/stable/more_types.html#function-overloading) to distinguish between the two cases:


<br>

<script src="https://gist.github.com/pedrovhb/1f43ba2325c87893e3cc805b66fc5ca6.js?file=protocol_self_overload_1.py"></script>
<details>
  <summary><i>click here to see the snippet if the gist is down</i></summary>

```python
from __future__ import annotations

from abc import ABC
from typing import ClassVar, Generic, Literal, NoReturn, Protocol, TypeVar, overload

_OutputT = TypeVar("_OutputT")


class HasStdout(Protocol):
    """Protocol for objects that store the stdout buffer in memory."""

    keep_stdout_buffer: ClassVar[Literal[True]]


class NoHasStdout(Protocol):
    """Protocol for objects that do not store the stdout buffer in memory."""

    keep_stdout_buffer: ClassVar[Literal[False]]


class MyClassBase(Generic[_OutputT], ABC):
    keep_stdout_buffer: ClassVar[Literal[True, False]]
    stdout_buffer: bytearray = bytearray()

    @overload
    def get_stdout(self: HasStdout) -> bytes:
        ...

    @overload
    def get_stdout(self: NoHasStdout) -> NoReturn:
        ...

    def get_stdout(self):
        """Returns the process' stdout if it is stored in memory; raises an exception otherwise."""
        if self.keep_stdout_buffer:
            return bytes(self.stdout_buffer)
        else:
            raise Exception(
                f"The stdout buffer is not kept for" f" class {self.__class__.__name__}"
            )

    def on_process_exit(self) -> _OutputT:
        """Subclasses should override this method to return the output of the process."""
        raise NotImplementedError
```
</details>
<br>


This way, we can get the best of both worlds - the user can choose whether to keep the buffer or not, and we can avoid runtime exceptions by letting mypy catch them for us.

<br>
<script src="https://gist.github.com/pedrovhb/1f43ba2325c87893e3cc805b66fc5ca6.js?file=protocol_self_overload_2.py"></script>
<details>
  <summary><i>click here to see the snippet if the gist is down</i></summary>

```python
class SubclassWithBuf(MyClassBase[str]):
    keep_stdout_buffer: ClassVar[Literal[True]] = True

    def on_process_exit(self) -> str:
        # Ok - type checks out!
        return self.get_stdout().decode()


class SubclassWithoutBuf(MyClassBase[str]):
    keep_stdout_buffer: ClassVar[Literal[False]] = False

    def on_process_exit(self) -> str:
        # Error - NoReturn has no attribute "decode"
        return self.get_stdout().decode()


reveal_type(SubclassWithBuf().get_stdout())
# Revealed type is 'builtins.str'

reveal_type(SubclassWithoutBuf().get_stdout())
# Revealed type is '<nothing>'
```
</details>
<br>


I'm still not certain whether this is a good idea or not.
On one hand, it avoids the need to create separate base classes, which can avoid a lot of complexity down the line if I decide that I want further subclasses of this.
On the other hand, it is a bit of a contrived idea for something that isn't particularly complex.
It's also an eyesore to have to annotate the class variable with `ClassVar` and `Literal` instead of just assigning a value and letting mypy infer the type.
It may be possible to do that, but I haven't figured out how.
If I just assign a `False` or `True` value in the subclass' body, mypy will infer the variable's type as `bool` and won't consider it to be a literal, so the overloads won't function as expected.
Still, it's a neat trick, and I thought I'd share it.
It's possible that there's similar situations where this could be useful, and I'll keep it in mind for the future.
