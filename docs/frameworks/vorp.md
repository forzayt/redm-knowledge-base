---
title: VORP Core
category: Frameworks
tags:
  - vorp
  - framework
  - redm
framework:
  - vorp
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/
  - https://docs.vorp-core.com/api-reference/lib
  - https://vorp-core.com/
  - https://github.com/VORPCORE/RDR3natives
---

# VORP Core

VORP Core is an open RedM framework with docs, a core API, and a modular library layer.

## Useful documentation areas

- introduction and getting started
- library import rules
- modular helpers for entities, blips, inputs, raycasts, prompts, points, zones, events, streaming, and logging
- inventory, character, menu, progress bar, and metabolism pages

## Key implementation notes

- the library uses module imports through `shared_script "@vorp_lib/import.lua"`
- encrypted assets cannot be imported through the lib layer
- the documentation highlights modules for client, server, and shared use

