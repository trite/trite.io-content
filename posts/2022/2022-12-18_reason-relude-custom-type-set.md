---
layout: post
title: Using Relude's `Set.t` functionality
created: 2022-12-18
---

While working on [Day 09](https://adventofcode.com/2022/day/9) of [Advent of Code](https://adventofcode.com) in ReasonML with [Relude](https://github.com/reazen/relude) as a standard library and [Melange](https://github.com/melange-re/melange) to compile I found it a bit painful to figure out how to use `Set` with a custom type structure.

# How to use `Relude.Set` in ReasonML

You need to tell `Set` how to compare items in order for it to be able to properly order its items and guarantee they are distinct. This can be done by creating a custom type and leveraging `Set.WithOrd` to wire up the comparison and equality functionality like so:

```reason
module Position = {
  type t = (int, int);

  let compare = ((x1, y1), (x2, y2)) =>
    switch(Int.compare(x1, x2)) {
    | `equal_to => Int.compare(y1, y2)
    | other => other
    };

  let eq = (t1, t2) =>
    compare(t1, t2) == `equal_to;

  module Ord = {
    type nonrec t = t;
    let compare = compare;
    let eq = eq;
  }
  module Set = Set.WithOrd(Ord);
};
```

Once that's done you can create an empty set and add items to it quite easily:

```reason
Position.Set.empty
|> Position.Set.add((0, 0))
|> Position.Set.add((12, 34))
|> Position.Set.add((42, 42))
|> Position.Set.add((0, 0))
|> Position.Set.toArray
|> Js.log;
```

Running this will result in a set with only 3 items in it, despite adding to it 4 times, since `(0, 0)` is added twice:

```shell
$ node _build/default/src/PositionSetTest.bs.js
[ [ 0, 0 ], [ 12, 34 ], [ 42, 42 ] ]
```