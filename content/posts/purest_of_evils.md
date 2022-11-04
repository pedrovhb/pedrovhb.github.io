---
title: "The purest of evils"
date: 2022-11-04T12:57:16-03:00
draft: false
------------


So I was playing with custom import functionality, and...

```python
def slightly_wrong(module_wrapper: ModuleWrapper, name: str, item: _T) -> _T:

    if callable(item):
        fn = cast(Callable[_P, _R], item)

        @functools.wraps(fn)
        def wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _R:
            result = fn(*args, **kwargs)
            if isinstance(result, float):
                result += result * random.uniform(-0.01, 0.01)
            return result

        return wrapper

    if isinstance(item, float):
        return item + item * random.uniform(-0.01, 0.01)

    return item


with import_hook(slightly_wrong):
    from math import pow, pi

# ...

print(pow(2, 3))  # 7.9625488355046645
print(pi)         # 3.1364642964169875
```

*No mathematicians were harmed in the making of this post.*
