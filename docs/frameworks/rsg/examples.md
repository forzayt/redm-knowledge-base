---
title: RSG Examples
category: Frameworks
tags:
  - rsg
  - examples
  - framework
framework:
  - rsg
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://redm.nativescripts.com/docs/scripts/redm/ns-lib
---

# RSG Examples

```lua
local lib = exports['ns-lib']
local player = lib:GetPlayer(source)
if not player then return end

local money = lib:GetMoney(source, "cash")
```

## Good examples to keep here

- player lookup
- item and money access
- framework-safe wrappers used across multiple resources

