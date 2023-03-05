---
title: Basics of relude-fetch library in ReasonML
created: 2023-03-05
---

# Why?

Finally creating a site to house the content that lives in my [trite.io-content](https://github.com/trite/trite.io-content) repository. The end goal is to build a javascript app via ReasonML that will fetch its actual content through Github API calls to the `trite.io-content` repo.

One important step for this means using [the fetch api](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). This can mean creating bindings by hand from Reason or using the [bs-fetch](https://github.com/reasonml-community/bs-fetch) library. Fortunately I like working with the [Relude](https://github.com/reazen/relude) standard library already, and there's an accompanying library [relude-fetch](https://github.com/reazen/relude-fetch) which makes it easy to work with fetch in a more idiomatic functional programming way!

# The problem: promises don't compose well

Working with fetch via direct bindings means managing javascript promises:

```
TODO: fetch binding example goes here when I find where I left it
```

The `bs-fetch` library takes care of a lot of the basics, but leaves us with `promises` as the main method of interaction:
```ocaml
Js.Promise.(
  Fetch.fetch("/api/hellos/1")
  |> then_(Fetch.Response.text)
  |> then_(text => print_endline(text) |> resolve)
);
```

TODO still: Promises are ok, but they can be a bit unwieldy and lack some expressiveness that can be attained through the power of `MONADS` (insert `amaze` face here?)


# The solution: IO monad

`relude-fetch` takes the `fetch` promise and converts it to a `Relude.IO`. This takes something that is composes clumsily and turns it into a compositional powerhouse through the power of monads!

[Here](https://github.com/reazen/relude-fetch/blob/master/src/ReludeFetch.re#L9) is where `relude-fetch` takes the [bs-fetch](https://github.com/reasonml-community/bs-fetch) bindings and turns them up to 11:
```ocaml
  let fetch: string => Relude.IO.t(Fetch.response, Js.Promise.error) =
    url =>
      Relude.IO.suspendIO(() => Relude.Js.Promise.toIO(Fetch.fetch(url)));
```



```ocaml
module Error = {
  type t = ReludeFetch.Error.t(string);
  let show = error => ReludeFetch.Error.show(a => a, error);
  module Type = {
    type nonrec t = t;
  };
};

module IOE = IO.WithError(Error.Type);
open IOE.Infix;

let fetchString = uri => ReludeFetch.fetch(uri) >>= ReludeFetch.Response.text;
```

# More reading
* [async/await is just the do-notation of the Promise monad](https://gist.github.com/peter-leonov/c86720d1517235a1f28cd453a9d39bb4)
* [Promises are Almost Monads](https://siawyoung.com/promises-are-almost-monads)