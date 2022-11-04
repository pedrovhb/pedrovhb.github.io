---
title: "Neat mypy trick - avoid runtime exceptions with Protocol and ClassVar"
date: 2022-09-26T11:23:00-03:00
draft: false
------------

I'm writing a little utility library that among other things deals with asynchronously managing subprocesses.
One of the things I want it to do is to make it easier to process output for arbitrary command line utilities.
It's still early on in development, so I'm in the fun part of the process of playing with concepts, trying things out and deciding what the API should look like.
That's just before the terrifying part of having to pick a name.
Anyways, the idea at this time is that to wrap command line tools, you'd subclass the library's provided base class and implement the relevant methods (say, doing something for each line of output).
By implementing a subclass, you're getting some base class functionality like async iteration and events for free.


Some CLI programs will output lines of text that are useful as streams, and others really only make sense to look at when the process is finished.
Compare `du` and `jpeg-recompress` for example.


With `du`, the output is a stream of lines that each carry some information that can be acted on immediately.
It could be useful to start processing them asynchronously as soon as they're available, instead of waiting for the process to finish.

```bash
$ du -h

56	./.local/pipx/shared/lib/python3.10/site-packages/pkg_resources/_vendor/importlib_resources/__pycache__
96	./.local/pipx/shared/lib/python3.10/site-packages/pkg_resources/_vendor/importlib_resources
20	./.local/pipx/shared/lib/python3.10/site-packages/pkg_resources/_vendor/jaraco/text/__pycache__
36	./.local/pipx/shared/lib/python3.10/site-packages/pkg_resources/_vendor/jaraco/text
28	./.local/pipx/shared/lib/python3.10/site-packages/pkg_resources/_vendor/jaraco/__pycache__
...
```
<br>


With `jpeg-recompress`, I don't really care about intermediate output - I just want to know when the process is done and what the final result was -

```bash
$ jpeg-recompress input.jpg output.jpg
Metadata size is 0kb
ssim at q=67 (40 - 95): 0.999779
ssim at q=81 (68 - 95): 0.999941
ssim at q=74 (68 - 80): 0.999995
ssim at q=70 (68 - 73): 0.999848
ssim at q=72 (71 - 73): 0.999897
Final optimized ssim at q=73: 0.999832
New size is 71% of original (saved 140 kb)
```
<br>


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


We can define a `Protocol` containing this class variable, and then use it as the `self` argument type for [method overloads](https://mypy.readthedocs.io/en/stable/more_types.html#function-overloading) to distinguish between the two (or more, in some other situation/abstraction) cases.
Here's some code with the relevant ideas:


<br>

<script src="https://gist.github.com/pedrovhb/1f43ba2325c87893e3cc805b66fc5ca6.js?file=protocol_self_overload_1.py"></script>
<details>
  <summary><i>click here to see the snippet if the gist is inaccessible</i></summary>

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
  <summary><i>click here to see the snippet if the gist is inaccessible</i></summary>

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
On one hand, it avoids the need to create separate base classes, which can avoid a lot of complexity down the line if I decide that I want to further subclass `MyClassBase`;
 it'd be nice to avoid having to deal with mixins and multiple inheritance, and having two different children subclasses for each parent class would quickly get out of hand.
On the other hand, it is a bit of a contrived idea for something that isn't particularly complex.
It's also an eyesore to annotate the class variable with `ClassVar` and `Literal` instead of just assigning a value and letting mypy infer the type.
If I just assign a `False` or `True` value in the subclass' body, mypy will infer the variable's type as `bool` and won't consider it to be a literal, so the overloads won't function as expected.
It may be possible to make that work (let me know if you have an answer), but I haven't figured out how to do it.
Still, it's a neat trick, and I thought I'd share it.
It's possible that there's similar situations where this could be useful, and I'll keep it in mind for the future.
