---
title: Native Scripts ns-lib
category: Frameworks
tags:
  - ns-lib
  - abstraction
  - redm
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

# Native Scripts ns-lib

`ns-lib` is a cross-framework abstraction layer for RedM resources.

## Supported systems

- frameworks: VORP, RSG-Core, RedEM:RP
- inventory: `ox_inventory`, `vorp_inventory`, `rsg-inventory`, `redemrp_inventory`
- SQL: `oxmysql`, `mysql-async`
- notifications: `ns-notify` and several fallback layers

## Why it matters

- one dependency line can support multiple frameworks
- it standardizes player, inventory, money, blip, ped, and teleport helpers
- it reduces repeated glue code across resources

