---
title: Melange (ReasonML), OCaml LSP, and VSCode
created: 2022-06-10
---

This post is gonna be quick. It's a list of things to check if you're having problems getting the OCaml LSP to work with your text editor (namely VSCode, but may apply to others).

![All kinds of sadness](img/ocamllsp-issues-wrong-version.png)

# Check that you launched from `esy`
Launching an app via `esy` allows it to run in the context of the appropriate [sandbox environment](https://esy.sh/docs/en/concepts.html#project-sandbox). If you aren't sure if the app was launched from esy, you can kill it and start it back up.

Example usage - run `esy [app] [args]` from the root of your project, where your `esy.json` file lives:
```sh
trite@DESKTOP-0ACBTNR:/mnt/c/git/hackerrank-melange$ esy code .
```

# Double check your LSP version
At the time of writing this (2022-06-10) the OCaml LSP requires use of a specific version to correctly work with Melange. This should change soon. 

<details><summary>@mayhewluke on the ReasonML discord provided a nice explanation</summary>
<p>
> Dune 3.0 provides an RPC server for language services to get the Merlin info from instead of writing to actual .merlin files, so ocaml-lsp 1.9.0 removed support for reading .merlin in favor of talking to Dune directly. Melange has to use .merlin for various reasons, so we used to pin LSP to 1.8.3, but 1.8.3 doesn't work with ocaml 4.14, so when melange updated to 4.14 language support broke.
> 
> @anmonteiro whipped up the --fallback... flag a few weeks ago, I think, but we don't yet have a release with it. A release has been requested, and apparently might land some time next week.
> https://discord.com/channels/436568060288172042/946816636197929080/984506621382250527 <== previous discussion on the topic

[Link to the original comment](https://discord.com/channels/235176658175262720/825155604641218580/984837516194635786) <== You may need to have joined the ReasonML discord for the link to work. [Link to the ReasonML Discord](https://discord.gg/reasonml)
</p>
</details>
