---
title: VORP Examples
category: Frameworks
tags:
  - vorp
  - examples
  - framework
framework:
  - vorp
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/core
  - https://docs.vorp-core.com/api-reference/lib
---

# VORP Examples

```lua
local Core = exports.vorp_core:GetCore()
local user = Core.getUser(source)
if not user then return end

local character = user.getUsedCharacter
local inventory = exports.vorp_inventory:getInventoryItems()
```

## Good examples to keep here

- character lookup
- inventory lookup
- prompt or blip helper usage
- small reusable wrappers for common VORP actions

