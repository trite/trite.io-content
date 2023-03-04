---
title: steps to use the melange-opam-template git template
created: 2023-03-03
---

Pre-requisites:
* Need [OPAM](https://opam.ocaml.org/) and [NPM](https://www.npmjs.com/) installed
* Need [make](https://www.gnu.org/software/make/) installed
* If using VSCode you'll need the `OCaml Platform` extension installed
* Maybe other things depending on your OS. Watch for errors during the `make init` step below. If you're missing a pre-requisite it'll show up in those errors

Steps to take:
* Create your own repo from the template here: https://github.com/melange-re/melange-opam-template by clicking `Use this template` => `Create a new repository`.
* Clone the repository to your local machine.
* Do a project-wide search/replace:
  - Search for all instances of `melange-opam-template`.
  - Replace with your project name, ex: `my-project`. I had issues with names containing a dot (`.`). If you use a dot in the name and these steps do not work that may be why.
* Rename the `melange-opam-template.opam` file in the root of the project to your project name while keeping the `.opam` file extension, ex: `my-project.opam`.
* **Important** - Commit your changes or the next step will ignore said changes and things likely won't work as intended.
* From a terminal in the root of your project run `make init`. This step may take around 5-15 minutes depending on your particular environment, but it only needs to be done when setting up the repo on a new machine.
* You may need to run `eval $(opam env)` at this point, pay attention for a message about it in the last few lines of output from the `make init` command. It's also fine to just run it at this point if you're not sure.
* If you're using VSCode make sure to close it and launch it from the terminal by running `code .` from the root of your project.
* Run `make watch` in one terminal to build and then watch for changes.
* If using VSCode and it's unhappy at this point try running `OCaml: Restart Language Server` from the command palette (`CTRL+SHIFT+P`/`CMD+SHIFT+P`).
* Run `make serve` in a separate terminal to start up the default React app locally.