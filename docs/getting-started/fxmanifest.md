---
title: fxmanifest.lua
category: Getting Started
tags:
  - fxmanifest
  - manifest
  - resource-metadata
framework:
  - redm
difficulty: beginner
last_updated: 2026-08-03
sources:
  - https://docs.fivem.net/docs/scripting-reference/resource-manifest/resource-manifest/
  - https://docs.fivem.net/docs/scripting-reference/resource-manifest/
---

# fxmanifest.lua

`fxmanifest.lua` declares how a resource loads, what game it targets, and which scripts or data files it exposes.

## Minimum RedM shape

```lua
fx_version 'cerulean'
game 'rdr3'

lua54 'yes'

client_scripts {
  'client/*.lua'
}

server_scripts {
  'server/*.lua'
}
```

## Notes

- use `game 'rdr3'` for RedM resources
- use `fx_version 'cerulean'` for modern manifests
- add data file entries when the resource streams meta files or other game data
- keep the manifest short and explicit

