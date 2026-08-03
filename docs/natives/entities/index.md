---
title: Entities Index
category: Natives
tags:
  - entities
  - world-state
  - networking
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://docs.fivem.net/docs/scripting-manual/entity-creation-and-ownership/
---

# Entities

Entities are the engine-level game objects that represent props, peds, animals, wagons, trains, weapons, pickups, and interactive scenery. This page focuses on `object` (scene props, pickups) and `vehicle` (wagons, carts) — for humans/animals see [peds/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/index.md) and [horses.md](file:///d:/Playground/redm-knowledge-base/docs/natives/horses.md).

## Sub-pages

| File | Content |
|---|---|
| [api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/entities/api.md) | Per-native signatures: creation, position, heading, visibility, networking, attachment, deletion |
| [examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/entities/examples.md) | Recipes: frozen prop, attach lantern to player belt, wagon + NPC driver, locked door, pickup-drop entity |

## Three Entity Types

| Engine kind | Creation native | Typical uses |
|---|---|---|
| **Object** | `CreateObject` / `CreateObjectNoOffset` | Benches, barrels, lanterns, crates, doors, signs, pickups |
| **Vehicle** | `CreateVehicle` / `CreateVehicleServerSetter` | Wagons (stagecoach, buggy, chuckwagon), trains, boats, mining carts |
| **Ped** | `CreatePed` / `CreatePedInsideVehicle` | Humans, horses, dogs, wildlife, player character |

### Shared natives across all three

Every entity responds to: `GetEntityCoords`, `SetEntityCoords`, `GetEntityHeading`, `SetEntityHeading`, `SetEntityRotation`, `GetEntityModel`, `DoesEntityExist`, `DeleteEntity`, `FreezeEntityPosition`, `SetEntityInvincible`, `SetEntityVisible`, `SetEntityCollision`, `AttachEntityToEntity`, `DetachEntity`, `NetworkGetNetworkIdFromEntity`.

## Entity Lifecycle (Ordered)

```lua
-- 1. Request the model (streaming)
local MODEL = `prop_barrel_02b`
RequestModel(MODEL)
while not HasModelLoaded(MODEL) do Wait(0) end

-- 2. Create
local obj = CreateObject(MODEL, x, y, z, true, false, false)  -- networked=true

-- 3. Configure
SetEntityAsMissionEntity(obj, true, false)
FreezeEntityPosition(obj, true)
PlaceObjectOnGroundProperly(obj)
SetEntityInvincible(obj, true)

-- 4. Use in gameplay...

-- 5. Destroy (clean up on script or map exit)
SetEntityAsMissionEntity(obj, false, true)
DeleteObject(obj)
SetEntityAsNoLongerNeeded(obj)
SetModelAsNoLongerNeeded(MODEL)
```

---

## Networking: Ownership vs Ghosts

A networked entity has exactly one "owner" client (the one that authored the last authoritative state update):

- **Owner clients:** Can call `SetEntity*` freely; updates propagate via 1 Hz state sync + 30 Hz culling.
- **Non-owners:** Any `SetEntity*` call will be reverted by the next owner sync. If you need to move something from a non-owner, emit a server event and have the owner apply it, or use `NetworkRequestControlOfEntity` first (best effort, not guaranteed).

Server can forcibly change ownership with:

```lua
-- server only
Entity(ent).state:set("owner_source", src, true)
-- or CFX:
NetworkMigrateEntityOwnership(ent, src)
```

## Attachment Pattern

Use `AttachEntityToEntity` to combine a prop with a ped's bone index (e.g. `BELT_LEFT`, `PH_R_Hand`).

| Bone index string | Ped bone name |
|---|---|
| `BELT_LEFT` | 40269 |
| `BELT_RIGHT` | 40918 |
| `PH_R_Hand` | 58866 |
| `SKEL_L_Hand` | 18905 |
| `IK_R_HEAD` | 12844 |

> Use `GetEntityBoneIndexByName(ped, "SKEL_L_Hand")` to translate at runtime — safer than hardcoded numbers across different models.

## Best Practices

1. **Don't create in a hot loop.** Throttled creation at 5–10 entities/frame minimum or you cause frame-time stutter.
2. **Pair every RequestModel with SetModelAsNoLongerNeeded.** Otherwise pool size grows unbounded and the client will stall.
3. **Use `SetEntityAsMissionEntity(obj, true, false)` right after create.** This prevents the game from culling your custom entities when a player leaves the cell.
4. **Explicitly destroy on `onResourceStop`.** A small `created = {}` table with a destructor loop is the simplest way.
5. **Use net IDs for IPC.** Never send entity handles across the network — they are local-only integers. Translate via `NetworkGetNetworkIdFromEntity(e)` on sender, `NetToObj(id)` on receiver, guarded by `NetworkDoesNetworkIdExist`.
