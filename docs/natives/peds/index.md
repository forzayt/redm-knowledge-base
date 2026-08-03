---
title: Peds Index
category: Natives
tags:
  - peds
  - entities
  - ai
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://github.com/femga/rdr3_discoveries
---

# Peds

Peds ("pedestrians") are the biped entities covering humans, wildlife, horses, dogs, and the local player character. This page covers human/animal ped spawning, configuration, combat, and clothing. Horse-specific mount/stamina/bonding natives live in [horses.md](file:///d:/Playground/redm-knowledge-base/docs/natives/horses.md); ped task AI live in [tasks/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/tasks/index.md).

## Sub-pages

| File | Content |
|---|---|
| [api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/api.md) | Per-native signatures for spawn, health/armour, damage, combat, relationships, clothing/props, speech |
| [examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/examples.md) | Recipes: vendor NPC with ambient speech, hostile bandit ambush, docile deer herd, deputy on duty with sheriff hat |

## Ped Types (first arg to `CreatePed`)

The ped-type integer controls initial relationship group defaults and combat-tuning presets. For new code just pass the right integer for the entity class you are spawning:

| Type | Family | Example |
|---|---|---|
| `4`  | Animal | `A_C_Deer_01`, `A_C_Wolf_01` |
| `26` | Civilian male | `S_M_M_ValShopKeeper_01` |
| `27` | Civilian female | `S_F_M_ValTownFolk_01` |
| `28` | Cop / law | `S_M_M_SheriffDeputy_01` |
| `29` | Gang male | `G_M_M_UniOwanDell_01` |
| `30` | Gang female | (rare) |
| `31` | Story male | `U_M_M_*` |
| `32` | Story female | `U_F_M_*` |
| `33` | Human MP male | `mp_male` |
| `34` | Human MP female | `mp_female` |
| `35` | Giant | `S_M_Y_AvatarGiant_01` |
| `36` | Creature | `A_C_Cougar_01`, `A_C_Bear_01` |

In practice the game will auto-correct many of these; most community scripts just pass `26` for human / `4` for animal.

## Relationship Groups (Law, Hostile, Neutral)

Relationship groups determine "who shoots whom" and flee behaviour. Use them instead of per-ped tuning.

```lua
AddRelationshipGroup("LAW")
AddRelationshipGroup("OUTLAWS")
AddRelationshipGroup("DEER")

SetRelationshipBetweenGroups(0, `LAW`, `OUTLAWS`)     -- Respect (don't fight)
SetRelationshipBetweenGroups(5, `OUTLAWS`, `LAW`)     -- Hate (fight)
SetRelationshipBetweenGroups(5, `LAW`, `OUTLAWS`)     -- Hate bidirectional
SetRelationshipBetweenGroups(5, `OUTLAWS`, `PLAYER`)
SetRelationshipBetweenGroups(3, `DEER`, `PLAYER`)      -- Fear = flee
```

`SetPedRelationshipGroupHash(ped, group)` applies it to a specific ped.

## Quick NPC Config (Vendor pattern)

90% of stationary vendor NPCs use exactly this set of natives:

```lua
RequestModel(M)
while not HasModelLoaded(M) do Wait(0) end
local ped = CreatePed(26, M, x, y, z, heading, false, false)
SetEntityAsMissionEntity(ped, true, false)
FreezeEntityPosition(ped, true)
SetEntityInvincible(ped, true)
SetBlockingOfNonTemporaryEvents(ped, true)   -- ignore gunfire/panic around it
SetPedCanBeDraggedByAnyVehicle(ped, false)
SetPedCanBeTargetted(ped, false)
SetPedCanLosePropsOnDamage(ped, false, 0)
SetPedFleeAttributes(ped, 0, 0)               -- never flee
SetPedCombatAttributes(ped, 0, false)         -- never enter combat
SetPedCombatAttributes(ped, 17, true)         -- BF_CanBeDraggedOutOfVehicle = no
SetPedKeepTask(ped, true)                     -- do not change tasks due to idle AI
SetModelAsNoLongerNeeded(M)
```

Then use tasks from [tasks/examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/tasks/examples.md) to give the NPC a scenario or dialogue.

## Best Practices

1. **Always validate the ped handle** returned from `CreatePed` (it can be 0 if the server rejected the spawn, or model didn't load in time).
2. **Use `SetEntityAsMissionEntity(ped, true, false)` immediately** after spawn — otherwise the game will despawn it when the player leaves the area.
3. **Pair every `CreatePed` with `DeletePed` + `SetEntityAsNoLongerNeeded`** in `onResourceStop`.
4. **For networked lawmen, spawn on the server side via event**, not client-side. Each client would spawn a duplicate; server spawns once and replicates.
5. **Use relationship groups, not per-ped combat flags**, for outlaw/law behaviour. Groups are orders of magnitude easier to tune than 50 individual `SetPedCombatAttributes` calls.
