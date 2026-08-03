---
title: RedEM Examples
category: Frameworks
tags:
  - redem
  - examples
  - framework
framework:
  - redem
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://redm.nativescripts.com/docs/scripts/redm/ns-lib
---

# RedEM Examples

```lua
local lib = exports['ns-lib']
if lib:HasItem(source, "empty_bottle", 1) then
  lib:Notify(source, "You have the item", "success")
end
```

## Good examples to keep here

- role-based inventory checks
- status integrations
- simple compatibility wrappers

