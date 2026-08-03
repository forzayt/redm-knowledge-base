---
title: Blip Examples
category: Natives
tags:
  - blips
  - examples
  - map
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/lib
---

# Blip Examples

```lua
local map = Import("blips").Map

local marker = map.Blips:Create({
  coords = vector3(0.0, 0.0, 0.0),
  sprite = "blip_ambient_sheriff",
  label = "Sheriff"
})

marker:SetScale(0.8)
```

## Notes

- use radius blips for search areas
- use attached blips for moving targets

