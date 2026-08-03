---
title: Native Overview
category: Natives
tags:
  - natives
  - reference
  - redm
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://natives.lawless-street.fr/
  - https://github.com/alloc8or/rdr3-nativedb-data
  - https://alloc8or.re/rdr3/nativedb/
  - https://github.com/nativewrappers/rdr3-natives
  - https://github.com/VORPCORE/RDR3natives
  - https://github.com/femga/rdr3_discoveries
---

# Native Overview

RedM / RDR3 "natives" are the low-level C functions the game engine exposes to Lua scripts. They are referenced either by name (`BlipAddForCoords`) or by their canonical `0x` hash (e.g. `0x554D9D53F696D002`), and are the primitive building blocks for every resource.

The official VORP / community docs live at **https://docs.redm.store/natives** and there are browsable databases at:

- https://alloc8or.re/rdr3/nativedb/ — de-facto lookup (by name / hash / namespace)
- https://natives.lawless-street.fr/ — search UI with multi-language signature view
- https://github.com/alloc8or/rdr3-nativedb-data — machine-readable JSON corpus (primary source)

## How Natives Work

Every native has a:

| Property | Meaning |
|---|---|
| **C signature** | Canonical prototype, e.g. `Blip BLIP_ADD_FOR_COORDS(Hash styleHash, float x, float y, float z)` |
| **Build hash** | `0x` hex used at runtime (e.g. `0x554D9D53F696D002`) |
| **Lua alias** | Human-friendly wrapper the client auto-imports (e.g. `BlipAddForCoords`) when a match is registered |
| **Namespace** | Broad family such as `HUD`, `PED`, `ENTITY`, `TASK`, `STREAMING`, `AUDIO`, `CFX` |
| **API set** | `client` (default, runs locally) or `server` (rare — only CFX ones like `GetPlayers`) |

### Three Ways to Call the Same Native

```lua
-- 1. Lua alias (preferred when available)
local blip = BlipAddForCoords(`BLIP_STYLE_SHOP`, -312.0, 768.0, 118.0)

-- 2. Explicit hash via Citizen.InvokeNative (for unaliased / unknown names)
local blip = Citizen.InvokeNative(0x554D9D53F696D002, `BLIP_STYLE_SHOP`, -312.0, 768.0, 118.0)

-- 3. Helper resource wrapping (redm-natives / redm_natives exports)
local n = exports['redm-native']:GetNative('BlipAddForCoords')
local blip = n.BlipAddForCoords(`BLIP_STYLE_SHOP`, -312.0, 768.0, 118.0)
```

> **Hash vs backtick:** The `` `HASH_NAME` `` syntax is a Lua string literal that RedM's parser automatically converts to the `joaat()` hash at parse-time. Always use backtick literals instead of `GetHashKey("HASH_NAME")` inside hot loops.

### The joaat (Jenkins one-at-a-time) Hash

RedM uses the same string-hashing convention as GTA V / FiveM: **Case-insensitive, underscores/dashes are stripped, then Jenkins one-at-a-time (joaat) is applied.**

```lua
-- These are equivalent:
joaat("BLIP_STYLE_SHOP")
`BLIP_STYLE_SHOP`
0xB44A118A  -- computed hash
```

Use `` `LITERAL` `` syntax for anything referenced in the R* string tables (styles, modifiers, control hashes, audio names, animation dicts, model names, blip sprites…).

---

## Namespace Map (Popular Families)

This docs project splits natives by *usage domain* rather than strict namespace. Here is how the domain folders map to engine namespaces:

| Folder | Engine namespaces | Typical use |
|---|---|---|
| [blips/](file:///d:/Playground/redm-knowledge-base/docs/natives/blips) | `HUD` + `MAP` (CFX) | `BlipAddForCoords`, `BlipAddModifier`, `SetBlipSprite`, `SetBlipName`… |
| [entities/](file:///d:/Playground/redm-knowledge-base/docs/natives/entities) | `ENTITY`, `OBJECT` (CFX) | `CreateObject`, `AttachEntityToEntity`, `SetEntityCoords`, `DeleteEntity`… |
| [peds/](file:///d:/Playground/redm-knowledge-base/docs/natives/peds) | `PED` (CFX) | `CreatePed`, `SetPedRelationshipGroupHash`, `SetEntityInvincible`, `GetEntityHealth`… |
| [tasks/](file:///d:/Playground/redm-knowledge-base/docs/natives/tasks) | `TASK` | `TaskGoToEntity`, `TaskCombatPed`, `TaskStartScenarioInPlace`, `ClearPedTasksImmediately`… |
| [streaming/](file:///d:/Playground/redm-knowledge-base/docs/natives/streaming) | `STREAMING` | `RequestModel`, `HasModelLoaded`, `SetModelAsNoLongerNeeded`, `RequestAnimDict`… |
| [prompts/](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts) | `HUD` prompt group (undoc) | `PromptRegisterBegin`, `PromptSetControlAction`, `PromptSetText`, `PromptRegisterEnd`, `PromptSetEnabled`… |

A small number of natives live outside these folders (player, camera, audio, UI). Those are handled outside this project for now.

---

## Documentation Strategy (This Repo)

To keep retrieval and AI-assisted context high-quality:

1. **Small page per family or single native.** Prefer 30–300 line files with heavy inline code samples.
2. **Signature block at top.** Always include:
   - Lua alias + hash hex
   - namespace + api set
   - typed parameters + return
3. **Keep 2 patterns side-by-side.** Show both the named alias (clean) and the `Citizen.InvokeNative` form (works even when alias hasn't been added to CFX mapping).
4. **1 example of real usage.** Natives without a concrete example will confuse you later.
5. **Notes on ownership / cleanup.** Most non-CFX natives leak memory if you don't explicitly release them (models, blips, prompts, anims). Mention it.
6. **StateBags and network natives first.** In multiplayer, prefer `LocalPlayer.state.x` and CFX natives (`CreateObject` with `isNetwork = true`) over purely-local manipulation.

## Anti-Patterns

| Pattern | Why | Fix |
|---|---|---|
| `while not HasModelLoaded(m) do Wait(0) end` with no timeout | Infinite hang if model is invalid | Add 5–10 s timeout; fail safely |
| `GetHashKey("STRING")` inside `Wait(0)` loop | Computes hash every frame | Use `` `STRING` `` literal or precompute |
| Spawning `CreatePed` / `CreateObject` with `isNetwork=false` on server-side script | Doesn't replicate to any client | Use `isNetwork=true` + `NetToVeh/Ped/Obj` or spawn client-side + register via statebag |
| Calling `SetBlipCoords` every frame when coords barely change | Wasteful | Update only when moved > 0.5 units or on position event |
| Never calling `SetModelAsNoLongerNeeded` / `RemoveBlip` | Memory grows over session lifetime | Always pair request/release in `onResourceStop` |

## Where to Look Next

- [blips](file:///d:/Playground/redm-knowledge-base/docs/natives/blips/index.md) — map markers, sprites, modifiers
- [entities](file:///d:/Playground/redm-knowledge-base/docs/natives/entities/index.md) — objects, props, vehicles, attachments
- [peds](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/index.md) — humans, lawmen, animals, customization
- [tasks](file:///d:/Playground/redm-knowledge-base/docs/natives/tasks/index.md) — AI behaviour: combat, scenarios, movement
- [streaming](file:///d:/Playground/redm-knowledge-base/docs/natives/streaming/index.md) — model/anim/IPL loading pipelines
- [prompts](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/index.md) — E-to-interact prompt UI
- [horses](file:///d:/Playground/redm-knowledge-base/docs/natives/horses.md) — horse-specific natives (model ref, mount, saddle, stamina)
