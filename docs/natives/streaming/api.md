---
title: Streaming API
category: Natives
tags:
  - streaming
  - api
  - loading
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.fivem.net/docs/game-references/data-files/
  - https://docs.fivem.net/docs/scripting-reference/resource-manifest/resource-manifest/
  - https://docs.vorp-core.com/api-reference/lib
---

# Streaming API

## Common surfaces

- request model and collision data
- request IPL or other extra content references
- manage focus around the relevant world area
- wait for streaming assets before spawning dependent entities

## Design notes

- never spawn a dependent entity before its model is ready
- avoid repeated request calls in a hot loop
- pair requests with a clear completion check

