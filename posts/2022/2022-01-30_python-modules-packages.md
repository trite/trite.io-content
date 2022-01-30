---
title: Python modules and packages
created: 2022-01-30
---

It's time to start using modules in my AoC code. Unfortunately the AoC repo structure wasn't created with Python in mind, restructuring it is waiting for now.

# Repo layout
Currently the layout of my AoC code looks something like this:

```
advent-of-code
    2015
        Day01
            p1-and-p2.py
        Day02
            p1.py
            p2.py
        Day03
    2016
        Day01
        Day02
        Day03
    trite_lib
        common.py
```

AoC code organized by Year/Day/Part seems to work overall, but this structure doesn't lend itself well to Python's module/package system.

# Solution?
For now the solution I've gone with is appending the root of the AoC repo to `sys.path` before attempting imports:

```python
from os.path import dirname, realpath
import sys
sys.path.append(dirname(dirname(dirname(realpath(__file__)))))
from trite_lib import common
```

Several lines required for 1 import, and the `sys.path.append` line makes me really sad, but it'll work for now at least.

# Links/References
https://docs.python.org/3/tutorial/modules.html
