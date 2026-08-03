---
title: VORP Setup
category: Frameworks
tags:
  - vorp
  - setup
  - framework
framework:
  - vorp
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/introduction
  - https://docs.vorp-core.com/api-reference/lib
---

# VORP Setup

## Base requirements

- add the VORP core resource
- import the lib with `shared_script "@vorp_lib/import.lua"`
- keep dependent scripts after the core resource in server start order

## Notes

- VORP docs describe the framework as actively maintained and intended for RedM RP servers
- the lib docs emphasize module-based imports for Lua only

