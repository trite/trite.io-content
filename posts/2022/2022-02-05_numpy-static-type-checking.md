---
layout: post
title: NumPy and static type checking via Pyright
created: 2022-02-05
---

Spending some more time on [Advent of Code 2015 day 6 part 1](https://adventofcode.com/2015/day/6). As mentioned in [the last post](2022-02-03_python-pillow-vscode-workflow.md), the goal is to make an animation of each command being applied in sequence.

I have already [solved the base problem](https://github.com/trite/advent-of-code/tree/39bebc72c0c2c57135183cc09da4b72aca3e52f8/2015/Day06/haskell) using fairly primitive types. [NumPy](https://numpy.org/) and [Pillow](https://pillow.readthedocs.io/en/stable/) work together nicely, and numpy has excellent support for manipulating 2-D arrays.

# Type of "[something]" is partially unknown
One of the first things I wanted to do is experiment with the conversion between numpy's `ndarray` and pillow's `Image` types. This is also where the first type constraints will be a problem for the type checker ([Pyright in this case](https://github.com/microsoft/pyright)):

<div class="post-image">![blah](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/numpy-type-checking-partially-unknown.png)</div>

Here's that full error message in all its hideous glory (formatted somewhat):
```
Type of "asarray" is partially unknown
    Type of "asarray" is "
        Overload [
            (
                a: _SupportsArray[dtype[_SCT@asarray]] | Sequence[_SupportsArray[dtype[_SCT@asarray]]] | Sequence[Sequence[_SupportsArray[dtype[_SCT@asarray]]]] | Sequence[Sequence[Sequence[_SupportsArray[dtype[_SCT@asarray]]]]] | Sequence[Sequence[Sequence[Sequence[_SupportsArray[dtype[_SCT@asarray]]]]]]
                , dtype: None = ...
                , order: Literal['K', 'A', 'C', 'F'] | None = ...
                , *
                , like: _SupportsArray[dtype[Unknown]] | _NestedSequence[_SupportsArray[dtype[Unknown]]] | bool | int | float | complex | str | bytes | _NestedSequence[bool | int | float | complex | str | bytes] = ...
            )
            -> 
            ndarray[Any, dtype[_SCT@asarray]]

            , (
                a: object
                , dtype: None = ...
                , order: Literal['K', 'A', 'C', 'F'] | None = ...
                , *
                , like: _SupportsArray[dtype[Unknown]] | _NestedSequence[_SupportsArray[dtype[Unknown]]] | bool | int | float | complex | str | bytes | _NestedSequence[bool | int | float | complex | str | bytes] = ...
            )
            ->
            ndarray[Any, dtype[Any]]

            , (
                a: Any
                , dtype: dtype[_SCT@asarray] | Type[_SCT@asarray] | _SupportsDType[dtype[_SCT@asarray]]
                , order: Literal['K', 'A', 'C', 'F'] | None = ...
                , *
                , like: _SupportsArray[dtype[Unknown]] | _NestedSequence[_SupportsArray[dtype[Unknown]]] | bool | int | float | complex | str | bytes | _NestedSequence[bool | int | float | complex | str | bytes] = ...
            )
            ->
            ndarray[Any, dtype[_SCT@asarray]]

            , (
                a: Any
                , dtype: dtype[Any] | type | _SupportsDType[dtype[Any]] | str | Tuple[Any, int] | Tuple[Any, SupportsIndex | Sequence[SupportsIndex]] | List[Any] | _DTypeDict | Tuple[Any, Any] | None
                , order: Literal['K', 'A', 'C', 'F'] | None = ...
                , *
                , like: _SupportsArray[dtype[Unknown]] | _NestedSequence[_SupportsArray[dtype[Unknown]]] | bool | int | float | complex | str | bytes | _NestedSequence[bool | int | float | complex | str | bytes] = ...
            )
            ->
            ndarray[Any, dtype[Any]]
        ]
    "PylancereportUnknownMemberType
```

So what's going on here? Basically we're working with an [overloaded function/method](https://en.wikipedia.org/wiki/Function_overloading). Following the function definition for `numpy.asarray` (`F12` by default in VSCode) reveals the following function signatures:

<div class="post-image">![overloaded function signatures](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/numpy-type-checking-asarray-overloaded.png)</div>

Don't worry if both of these just seem like walls of garbage. There's a lot to unpack, hopefully this might help a bit:

<div class="post-image">![overloaded function signatures comparison](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/numpy-type-checking-asarray-overloaded-comparison.png)</div>

Left side of the above image shows the function definitions, right side shows the error message with formatting applied. What this effectively means is:

# The type checker needs more information
In order to guarantee [correctness](https://en.wikipedia.org/wiki/Correctness_(computer_science)) of the code the [type system](https://en.wikipedia.org/wiki/Type_system) needs some information. Combining strict typing with function overloading means type information must be more narrow. The type checker doesn't just have 1 function to compare against now, but 4; and there is very likely a lot of overlap between them.

# Solutions

A few solutions that come to mind:
 * Provide enough information so the type checker can determine which version of the function is being called.
   - This function signature would be pretty hard to narrow down with all the `Any` types it uses.
   - The code will likely be unreadably verbose this way, if it is even possible.
 * Ignore the UnknownMemberType error for this file.
   - Done by adding `#pyright: reportUnknownMemberType=false`
   - Provides poor type checking for the rest of the code in this file, would prefer something else.
 * **Ignore this line via `#type: ignore`**
   - This breaks very little type checking on its own, so it's reasonable.
   - Adding back some of the lost guarantees is relatively easy with this route.

Ignoring one line can work pretty well. It comes with drawbacks, and can add exponential complexity the more it is used.

# Potholes 
Time to analogize briefly. Picture a street (or 3):

<div class="post-image">![which would you rather travel?](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/numpy-type-checking-road-analogy.png)</div>

The first section of road is fully type checked. Traveling it requires `less mental effort` than the street with a few potholes, which in turn requires less than the third street. Ignoring even a single line like this moves us from street `1` to street `2` in the image:

```python
from PIL import Image
import numpy as np

im = Image.new('RGB', (1000, 1000), (0, 0, 0))
arr = np.asarray(im) # type: ignore
print(arr)
```

Now there's a pothole to watch out for. But it's just 1, and it's pretty easy to watch out for. Just don't do something like this and you're all set!

```python
import numpy as np

# im = Image.new('RGB', (1000, 1000), (0, 0, 0))
im = 'blah'
arr = np.asarray(im) # type: ignore
print(arr)
```

Of course each thing that gets ignored must now be tracked by you, the developer. That can add up much faster than most folks realize.

What if we could take advantage of being lazy and ignoring this complex type signature, but then add our own guarantees? Doable, though it is a bit like [using ramen](https://knowyourmeme.com/memes/fixing-things-with-ramen) to fill in the pothole: technically `a` solution, certainly not `the best` one.

# Carefully ignoring types
A simple wrapper function can be the ramen in this tortured analogy, we just have to mold it into the right shape:

```python
from PIL import Image
import numpy as np
import numpy.typing as npt

def asarray(img: Image.Image) -> npt.NDArray[np.uint8]:
    return np.asarray(img) # type: ignore

im = Image.new('RGB', (1000, 1000), (0, 0, 0))
arr = asarray(im)
print(arr)
```

This local `asarray` function now forces correct type usage in pyright! VSCode will yell at us when passing the wrong type to our function:

<div class="post-image">![asarray wrapper error](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/numpy-type-checking-wrapping-asarray-error.png)</div>

And works as intended with the proper type:

<div class="post-image">![asarray wrapper working](https://github.com/trite/trite.io-content/raw/main/posts/2022/img/numpy-type-checking-wrapping-asarray.png)</div>