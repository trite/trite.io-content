---
title: Adventures in Python static type checking
created: 2022-01-18
last modified: 2022-02-04
---

Coming back to Python this weekend for a job interview has gone from fun nostalgia to full-blown learning adventure. I haven't touched Python much in the last 3 years. My recent time has been spent heavily focusing on Functional Programming concepts, lots of Haskell and F#.

These are some notes and things of interest.

# Type Checker -- MyPy vs Pyright vs Pytype vs Pyre

[PEP 484](https://www.python.org/dev/peps/pep-0484/) gave Python support for static type checking. This is of course an opt-in feature, since Python's default type system is ~~dynamic~~ [duck](https://en.wikipedia.org/wiki/Duck_typing) *(quack)*. After several hours and 3 [advent of code](https://adventofcode.com/) problems I sat down and started looking at what would be involved.

It's really quite easy to get setup with either MyPy or Pyright. Pytype and Pyre are on the list to check out eventually.

## MyPy

http://mypy-lang.org/

Started to check out MyPy briefly, but the pattern matching added in Python 3.10 was not yet supported by it. This cut things pretty short.

## Pyright/Pylance

Pyright appears to be Microsoft's stab at type checking, while Pytype and Pyre look to be Google and Facebook's respectively:
* https://github.com/microsoft/pyright
* https://github.com/google/pytype
* https://github.com/facebook/pyre-check

Pylance is a VSCode extension providing lots of intellisense QoL and other neat features for developing Python:
https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance

### Activating typechecking in Pylance:
* Open the settings ui (command pallette -> `Preferences: Open Settings(UI)`)
* Search for `typeCheckingMode`
* Change `Python > Analysis: Type Checking Mode` to your preferred level, I'm using `strict`

Once activated you'll either be greeted by basically nothing changing if your code is already compliant, or by a spectacular wall of errors and warnings to fix. The wall of errors version is intimidating at first, but it should require considerably fewer code changes than it seems. Most of those changes are likely just type annotations, not modifying actual code behavior.

# TODO: expand on these later
* Use subclassing/inheritance, not type aliasing, to keep similar types in order
* Python not only has the concept of algebraic sum types (union, discriminated union, etc) but added FP style support for expressing them using pipes (`|`) instead of the Union[a,b,c] style. https://www.python.org/dev/peps/pep-0604/
* @dataclass decorator for quick/easy type guarantees
* TypeVar to declare generic types, has no purpose other than type checking https://docs.python.org/3/library/typing.html#typing.TypeVar
* Expand on this post: https://ncik-roberts.github.io/posts/pep622.html
  - Looks like Python treats the pipe in pattern matching as fully separating patterns, meaning each must implement the same guard if they all want to use it.
  - A better approach for this might be to remove the guard from the pattern matching portion of the code and add a direct if into the execution portion. Need to mess around with this.