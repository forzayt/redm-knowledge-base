---
title: Prompts API
category: Natives
tags:
  - prompts
  - api
  - interaction
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://redm.nativescripts.com/docs/scripts/redm/ns-lib
  - https://docs.vorp-core.com/api-reference/lib
---

# Prompts API

## Common surfaces

- direct native prompt creation and update calls
- framework helpers that wrap prompt groups
- reusable abstractions for context-sensitive UI

## Design notes

- keep prompt creation and teardown in the same module
- store prompt handles in one table per resource
- avoid duplicate registrations for the same key
- use prompt groups when several interactions belong together

