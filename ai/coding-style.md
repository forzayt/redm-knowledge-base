---
title: Coding Style
category: AI Rules
tags:
  - ai-rules
  - style
  - lua
framework:
  - vorp
  - rsg
  - redem
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://www.lua.org/manual/5.4/
  - https://www.lua.org/pil/contents.html
---

# Coding Style

- Prefer local variables over globals.
- Keep functions small and single-purpose.
- Use descriptive names for game entities and framework handles.
- Separate server, client, and shared logic.
- Keep examples copy-pasteable.

## Lua habits

- Use tables for structured data.
- Use metatables only when they clearly reduce repetition.
- Prefer event-driven flow over busy loops.
- Use coroutines only when the yield points are explicit and safe.

