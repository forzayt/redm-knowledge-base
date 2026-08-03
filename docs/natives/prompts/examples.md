---
title: Prompt Examples
category: Natives
tags:
  - prompts
  - examples
  - interaction
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/lib
  - https://redm.nativescripts.com/docs/scripts/redm/ns-lib
---

# Prompt Examples

```lua
local Prompts = Import("prompts").Prompts

local prompt = Prompts:Create({
  key = "open_shop",
  text = "Open Shop",
  control = "INPUT_CONTEXT",
  coords = vector3(0.0, 0.0, 0.0)
})

prompt:Start()
```

## Notes

- keep text short
- group related prompts by activity
- destroy prompts when the interaction no longer exists

