---
title: Resource Structure
category: Getting Started
tags:
  - resources
  - folder-structure
  - fxmanifest
framework:
  - redm
difficulty: beginner
last_updated: 2026-08-03
sources:
  - https://docs.fivem.net/docs/scripting-reference/resource-manifest/resource-manifest/
  - https://docs.fivem.net/docs/game-references/data-files/
---

# Resource Structure

RedM resources are easiest to maintain when each resource has a narrow purpose.

## Practical layout

- `client/` for client-side logic
- `server/` for server-side logic
- `shared/` for cross-side code
- `config/` for configuration tables
- `data/` for JSON or other structured inputs
- `docs/` for resource-specific usage notes

## Good practices

- keep one feature per resource when possible
- separate framework glue from game logic
- keep streaming assets and code separate
- add a clear manifest and dependency ordering

