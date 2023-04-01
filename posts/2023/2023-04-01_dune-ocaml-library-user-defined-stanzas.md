---
title: Dune build failure trying to make an OCaml library
created: 2023-04-01
---

# The Problem
Trying to learn more about ReasonML, OCaml, and Dune to better understand how to troubleshoot build issues and understand how all these tools work together.

To that end I have been working on creating an [ocaml project](https://github.com/trite/ocaml_coding_exercises) to use for coding exercises. An executable doesn't make the most sense to me for this kind of usage, so creating the project as a library. 

With a `src/dune` file set to the below snippet the build would fail. First the snippet:
```lisp
(include_subdirs qualified)

(library
 (name ocaml_coding_exercises))
```

And here's the error it produces:
```shell
opam exec -- dune build
Error: The package ocaml_coding_exercises does not have any user defined
stanzas attached to it. If this is intentional, add (allow_empty) to the
package definition in the dune-project file
-> required by _build/default/ocaml_coding_exercises.install
-> required by alias all
-> required by alias default
make: *** [Makefile:18: build] Error 1
```

Googling for `does not have any user defined stanzas attached to it` lead to [this page](https://discuss.ocaml.org/t/dune-the-package-xxx-does-not-have-any-user-defined-stanzas-attached-to-it/9859/2) on the ocaml discuss forums. The last comment turned out to be the key:


"Just ran into this. Very confusing error message. Would be much better if it mentioned the `public_name` issue directly." ([direct link to message](https://discuss.ocaml.org/t/dune-the-package-xxx-does-not-have-any-user-defined-stanzas-attached-to-it/9859/3))

Which in turn lead to the [library stanza reference](https://dune.readthedocs.io/en/stable/dune-files.html#library) on the dune docs. Looks like library project (as opposed to a library IN an application project) needs to have a public name specified to build.

# The Solution
Simply add a `public_name` stanza inside the `library` stanza like so:
```lisp
(include_subdirs qualified)

(library
 (name ocaml_coding_exercises)
 (public_name ocaml_coding_exercises))
```