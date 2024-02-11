---
layout: post
title: Using Emacs with Melange
created: 2022-10-07
---

Make sure you are on ocaml 4.14.0 or later.

For doom emacs:
- `SPC f p` to open the Doom config file dialog
- Select `config.el`
- Add `(setq lsp-ocaml-lsp-server-command '("esy" "ocamllsp" "--fallback-read-dot-merlin"))`
- Restart emacs entirely
  - Or: reload using `SPC h r r` then restart the LSP with `M-x` => `lsp-workspace-restart`

# Links
- [Discord thread](https://discord.com/channels/235176658175262720/1010255043397693500)
- [Discord channel](https://discord.com/channels/235176658175262720/825155604641218580)
- [Discord server](https://discord.gg/reasonml)
