---
title: Streaming Examples
category: Natives
tags:
  - streaming
  - examples
  - loading
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/lib
  - https://docs.fivem.net/docs/game-references/data-files/
---

# Streaming Examples

```lua
local model = joaat("u_m_m_valtownfolk_01")

RequestModel(model)
while not HasModelLoaded(model) do
  Wait(0)
end

-- spawn logic goes here
SetModelAsNoLongerNeeded(model)
```

## Notes

- always release assets when they are no longer needed
- keep streaming waits short and explicit

