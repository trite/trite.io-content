---
title: Basics of relude-fetch library in ReasonML
---

# Why?

Finally creating a site to house the content that lives in my [trite.io-content](https://github.com/trite/trite.io-content) repository. The end goal is to build a javascript app via ReasonML that will fetch its actual content through Github API calls to the `trite.io-content` repo.

One important step for this means using [the fetch api](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). This can mean creating bindings by hand from Reason or using the [bs-fetch](https://github.com/reasonml-community/bs-fetch) library. Fortunately I like working with the [Relude](https://github.com/reazen/relude) standard library already, and there's an accompanying library [relude-fetch](https://github.com/reazen/relude-fetch) which makes it easy to work with fetch in a more idiomatic functional programming way!

# The problem
To use fetch from ReasonML means either creating bindings by hand or using a library that has taken care of this already.

The `bs-fetch` library knocks out the basics, but leaves us with `promises` as the main method of interaction:

```reason
Js.Promise.(
  Fetch.fetch("/api/hellos/1")
  |> then_(Fetch.Response.text)
  |> then_(text => print_endline(text) |> resolve)
);
```

Promises are fine and all, but both JavaScript and ReasonML have better mechanisms for dealing with asynchronous behavior.

In JavaScript this usually means using async/await. In functional languages like ReasonML this often means using `monads`, which is what this post will focus on.

# The solution
`relude-fetch` takes the `bs-fetch` promise and converts it to use `Relude.IO`. This takes something that composes poorly and turns it into a compositional powerhouse!

[Here](https://github.com/reazen/relude-fetch/blob/master/src/ReludeFetch.re#L9) is where `relude-fetch` takes the [bs-fetch](https://github.com/reasonml-community/bs-fetch) bindings and turns them up to 11:

```reason
let fetch: string => Relude.IO.t(Fetch.response, Js.Promise.error) =
  url =>
    Relude.IO.suspendIO(() => Relude.Js.Promise.toIO(Fetch.fetch(url)));
```

So how exactly does one use this?

# Usage
Start by defining the type interface for errors. Doing this first will simplify later usage:

```reason
module Error = {
  type t = ReludeFetch.Error.t(string);
  let show = error => ReludeFetch.Error.show(a => a, error);
  module Type = {
    type nonrec t = t;
  };
};

module IOE = IO.WithError(Error.Type);
```

Once done you can then access the available infix operators to further simplify usage:

```reason
open IOE.Infix;
```

Now you have access to the monadic bind operator `>>=` which allows for easier function composition. For example one could write a simple function to take a uri as input and return either the page content as a string, or else an error:

```reason
let fetchString = uri => ReludeFetch.fetch(uri) >>= ReludeFetch.Response.text;
```

Here's what it looks like to use our newly created `fetchString` command:

```reason
uri
|> fetchString
|> IO.map(content => setter(_ => content))
|> IO.mapError(error => setter(_ => error |> ContentFetch.Error.show))
|> IO.unsafeRunAsync(Result.fold(() => (), () => ()));
```

Here is what's going on in the above code:
* The first line is our `uri` in string format.
* The second line (`fetchString`) applies our function, which lifts everything into an IO. This IO is an operation that is "waiting" to happen, but won't actually do anything just yet.
* The third line (`IO.map`) specifies what to do with the result if the fetch call was successful. In this case using a React state setter function to update some component with the response body.
* The fourth line (`IO.mapError`) specifies what to do with the error if the fetch call failed. This example would convert the error to a string and then display it using the same setter function.
* The fifth line (`IO.unsafeRunAsync`) finally runs the operation, causing the fetch call to fire and then handling either the result (`IO.map`) or failure (`IO.mapError`).

# More reading
* [async/await is just the do-notation of the Promise monad](https://gist.github.com/peter-leonov/c86720d1517235a1f28cd453a9d39bb4)
* [Promises are Almost Monads](https://siawyoung.com/promises-are-almost-monads)
* [relude-fetch test examples](https://github.com/reazen/relude-fetch/blob/master/__tests__/ReludeFetch_test.re)
* [relude-fetch usage demo](https://github.com/reazen/relude-fetch/tree/master/examples/demo)