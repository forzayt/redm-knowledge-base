---
title: Task Examples
category: Natives
tags:
  - tasks
  - examples
  - behavior
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://github.com/femga/rdr3_discoveries/blob/master/README.md
---

# Task Examples

```lua
-- Example shape only: keep task calls isolated per behavior family.
local ped = PlayerPedId()
-- task helper goes here
```

## Notes

- keep tasks near the entity they control
- stop repeating task setup inside every tick loop

