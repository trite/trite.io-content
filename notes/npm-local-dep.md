---
title: Reference a local npm package
created: 2022-06-17
---

If you have one npm project that references another and you need to point at a local build...

```json
  "dependencies": {
    "bs-bastet": "2.0.0",
    "relude": "file:/mnt/c/git/relude"
  }
```