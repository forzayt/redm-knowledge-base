---
title: Entity Examples
category: Natives
tags:
  - entities
  - examples
  - networking
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/lib
---

# Entity Examples

```lua
local Entities = Import("entities").Entities

local box = Entities.Object:Create({
  model = "p_cs_crate01x",
  coords = vector3(0.0, 0.0, 0.0),
  isNetworked = true
})

box:SetHeading(90.0)
```

## Notes

- keep creation and deletion in the same module
- track network IDs if another resource needs to reference the entity

