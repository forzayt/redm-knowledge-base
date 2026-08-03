---
title: Ped Examples
category: Natives
tags:
  - peds
  - examples
  - ai
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/lib
  - https://github.com/femga/rdr3_discoveries
---

# Ped Examples

```lua
local entity = Import("entities")

local ped = entity.Ped:Create({
  model = "u_m_m_valtownfolk_01",
  coords = vector3(0.0, 0.0, 0.0),
  heading = 0.0
})

ped:SetModel("u_m_m_valtownfolk_01")
```

## Notes

- keep one spawn helper per NPC family
- separate combat peds from ambient peds
- register cleanup for each spawned entity

