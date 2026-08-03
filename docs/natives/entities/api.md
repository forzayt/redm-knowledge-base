---
title: Entities API
category: Natives
tags:
  - entities
  - api
  - world-state
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/lib
  - https://docs.fivem.net/docs/scripting-manual/introduction/about-native-functions/
---

# Entities API

## Core concepts

- every spawned entity should have an owner and a cleanup plan
- use helper wrappers when the same entity logic repeats across features
- prefer explicit state tables over hidden globals

## Common entity families

- peds
- vehicles
- objects
- attached props

