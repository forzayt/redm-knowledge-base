---
title: Blips Index
category: Natives
tags:
  - blips
  - map
  - ui
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://github.com/femga/rdr3_discoveries/blob/master/useful_info_from_rpfs/blip_modifiers/README.md
---

# Blips

Blips are the game's HUD map markers. Unlike FiveM (which uses numeric sprite IDs), RedM blips are built from three composable layers:

1. A **style hash** (`BLIP_STYLE_*`) — the base layout and intended purpose
2. Zero or more **modifier hashes** (`BLIP_MODIFIER_*`) — overlay color, alpha, category, pulse, area behaviour, conditions
3. Optional **sprite override** (`BLIP_*`) + label via `SetBlipName` / `_SET_BLIP_NAME_FROM_PLAYER_STRING`

## Sub-pages

| File | Content |
|---|---|
| [api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/blips/api.md) | Per-native signatures + notes for all create/set/get/remove functions |
| [examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/blips/examples.md) | Recipes: POI array, ped-tracked blip, radius pulse, area rect, server netId-tracking, auto-cleanup |

## Style → Use Cheatsheet

| Style hash | Purpose |
|---|---|
| `BLIP_STYLE_AMBULANCE` | Doctor / medic |
| `BLIP_STYLE_AMBIENT_SHERIFF_STAR` | Law / sheriff station |
| `BLIP_STYLE_AMBIENT_SHACK_ICON` | Random stranger / shack |
| `BLIP_STYLE_ANIMAL` | Generic animal spawn |
| `BLIP_STYLE_BOUNTY` | Bounty target |
| `BLIP_STYLE_CAMP` | Gang / player camp |
| `BLIP_STYLE_DESTINATION` | Mission destination |
| `BLIP_STYLE_DINER` | Saloon / diner |
| `BLIP_STYLE_EMPTY_LEGEND` | Marker without a legend entry |
| `BLIP_STYLE_GANG` | Gang hideout |
| `BLIP_STYLE_HIDE_AND_SEEK_AREA` | Radius for hide-and-seek events |
| `BLIP_STYLE_HORSE` | Horse or stable |
| `BLIP_STYLE_MISSION_AREA` | Mission interior/area rect |
| `BLIP_STYLE_OBJECTIVE` | Main story / quest objective |
| `BLIP_STYLE_PICKUP` | Pickup / loot bag |
| `BLIP_STYLE_PLAYER` | Player/crew marker (auto-enables route rendering) |
| `BLIP_STYLE_POI` | Map-legend POI |
| `BLIP_STYLE_SHOP` | General store, gunsmith, tailor |
| `BLIP_STYLE_WAYPOINT` | User/script waypoint |
| `BLIP_STYLE_STAGECOACH` | Stagecoach stop |
| `BLIP_STYLE_TRAIN` | Train station |
| `BLIP_STYLE_MAIL` | Post office |
| `BLIP_STYLE_SHERIFF_CARAVAN` | Prisoner wagon / law caravan |

## Modifier → Use Cheatsheet (most common)

| Modifier | Effect |
|---|---|
| `BLIP_MODIFIER_MP_COLOR_1` … `MP_COLOR_32` | Recolors to one of the multiplayer legend colors |
| `BLIP_MODIFIER_AREA` | Turns base into an area/circle visual |
| `BLIP_MODIFIER_AREA_PULSE` | Smooth 4s pulse on area alpha |
| `BLIP_MODIFIER_AREA_DIRECTIONAL` | Adds edge pointer arrow for distant targets |
| `BLIP_MODIFIER_ATTENTION` | Pulsing "attention" overlay, white |
| `BLIP_MODIFIER_TOD_DAYTIME_ONLY` / `TOD_NIGHTTIME_ONLY` | TOD visibility gate |
| `BLIP_MODIFIER_COMPASS_OBJECTIVE` | Objective marker: mini-map direction, hides within ~35 m |
| `BLIP_MODIFIER_AREA_HIDE_ON_INSIDE` / `AREA_HIDE_ON_OUTSIDE` | Player-distance toggle visibility |
| `BLIP_MODIFIER_CULL_ON_DEATH` | Remove blip when its entity dies |
| `BLIP_MODIFIER_ENEMY` | Red threat colour + range behaviour |
| `BLIP_MODIFIER_COMPANION_WOUNDED` | Revive crosshair overlay |
| `BLIP_MODIFIER_DISTANCE_FADE_LONG` / `MEDIUM` / `SHORT` | Far-range fade profiles |

> **Rule of thumb:** Call `BlipAddModifier` BEFORE `SetBlipName` / `SetBlipSprite` — modifiers often reset legend labels and category.
