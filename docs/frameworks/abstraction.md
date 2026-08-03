---
title: Framework Abstraction
category: Frameworks
tags:
  - frameworks
  - abstraction
  - cross-framework
framework:
  - vorp
  - rsg
  - redem
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://redm.nativescripts.com/docs
  - https://redm.nativescripts.com/docs/scripts/redm/ns-lib
---

# Framework Abstraction

If you want one codebase to run across VORP, RSG, and RedEM, isolate the differences behind a single adapter layer.

## Preferred shape

- app logic calls one shared API
- adapter resolves framework objects, player records, inventory, and money APIs
- page-level docs explain the abstraction once, then point to framework-specific details

## Why this works

- fewer branching paths in gameplay code
- easier retrieval for AI systems
- simpler maintenance when a framework changes

