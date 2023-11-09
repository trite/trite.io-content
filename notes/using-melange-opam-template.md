---
title: steps to use the melange-opam-template git template
created: 2023-03-03
---

Pre-requisites:

- [OPAM](https://opam.ocaml.org/) installed
  - OPAM itself is installed with: `bash -c "sh <(curl -fsSL https://raw.githubusercontent.com/ocaml/opam/master/shell/install.sh)"`
  - Pre-requisites for OPAM will display when attempting `opam init` if missing
  - OPAM pre-reqs can be installed using your OS package manager. For example on Ubuntu 22.04: `sudo apt install build-essential unzip bubblewrap`
- [NPM](https://www.npmjs.com/) installed
  - I suggest using FNM for this:
    - FNM install: `curl -fsSL https://fnm.vercel.app/install | bash`
    - Install node via FNM: `fnm install 18.16.1`
    - Set a default node version if one isn't already set: `fnm default 18.16.1`
- If using VSCode you'll usually want the `OCaml Platform` extension installed
  - Most other major code editors in use today have OCaml support, check online for your specific editor

Setting up your code repository:

- Create your own repo from the template here: https://github.com/melange-re/melange-opam-template by clicking `Use this template` => `Create a new repository`. You can then clone the repo to your local machine
  - Alternately you can simply clone the melange-opam-template repo above into a project folder of your choosing
- Do a project-wide search/replace:
  - Search for all instances of `melange-opam-template`.
  - Replace with your project name, ex: `my-project`. I recommend avoiding dots (`.`) in the project name
- Rename the `melange-opam-template.opam` file in the root of the project to your project name while keeping the `.opam` file extension, ex: `my-project.opam`.
- **Important** - If you encounter issues in the following steps you may need to commit your changes at this point and then revisit those steps.
- From a terminal in the root of your project run `make init`. This step may take around 5-15 minutes depending on your particular environment, but it only needs to be done when setting up the repo on a new machine.
- You may need to run `eval $(opam env)` at this point, pay attention for a message about it in the last few lines of output from the `make init` command. It's also fine to just run it at this point if you're not sure.
- If you're using VSCode you may need to either relaunch it or run a `Reload Window` command at this point to get LSP functionality working.
- Run `make watch` in one terminal to build and then watch for changes.
- If using VSCode and it's unhappy at this point try running `OCaml: Restart Language Server` from the command palette (`CTRL+SHIFT+P`/`CMD+SHIFT+P`).
- Run `make serve` in a separate terminal to start up the default React app locally.
- Make something!
