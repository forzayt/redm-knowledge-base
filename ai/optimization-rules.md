---
title: Optimization Rules
category: AI Rules
tags:
  - ai-rules
  - optimization
  - performance
framework:
  - vorp
  - rsg
  - redem
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://redm.nativescripts.com/docs/scripts/redm/ns-lib
  - https://docs.vorp-core.com/api-reference/lib
---

# Optimization Rules

- Never poll every frame unless there is no event-driven alternative.
- Cache expensive lookups such as player ped, framework objects, and shared config.
- Clean up spawned entities, blips, prompts, and threads.
- Avoid duplicate registration of the same event or item.
- Prefer shared helpers over repeated framework-specific glue.

## RedM-specific notes

- Be careful with entity lifetimes.
- Be careful with network ownership.
- Use framework-aware abstractions when the same logic must work on VORP, RSG, and RedEM.

