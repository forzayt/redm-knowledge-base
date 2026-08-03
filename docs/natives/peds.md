---
title: Peds
category: Natives
tags:
  - peds
  - entities
  - spawning
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://natives.lawless-street.fr/
  - https://github.com/nativewrappers/rdr3-natives
  - https://github.com/femga/rdr3_discoveries
  - https://alloc8or.re/rdr3/nativedb/?n=CreatePed
---

# Peds

Peds (pedestrians / biped characters) are the most commonly-used entity type in RP servers. They represent civilians, lawmen, vendors, animals (Cow, Wolf, Deer), and the player's MP character (via `mp_male`, `mp_female`).

## Create a Ped

```lua
local MODEL = `S_M_M_VALSHOPKEEPER_01`
RequestModel(MODEL)
while not HasModelLoaded(MODEL) do Wait(0) end

local ped = CreatePed(
    26,                           -- pedType (26 = civilian male / CIVMALE. Value is often ignored; just match sex for anim variety)
    MODEL,                        -- model hash
    -308.0, 770.0, 118.5 - 1.0,  -- x y z
    180.0,                        -- heading
    false,                        -- isNetwork (false = local-only NPC)
    false                         -- bScriptHostPed (false unless you need pinned host ownership)
)
SetModelAsNoLongerNeeded(MODEL)

SetEntityInvincible(ped, true)
FreezeEntityPosition(ped, true)
SetBlockingOfNonTemporaryEvents(ped, true)  -- ignore ambient panic/combat
SetPedCanBeDraggedByAnyVehicle(ped, false)
SetPedFleeAttributes(ped, 0, 0)
SetPedCombatAttributes(ped, 46, true)       -- BF_CanFightArmedPedsWhenNotArmed
```

### Animals

Use the animal model hash directly and pass `4` as pedType (`PED_TYPE_ANIMAL`):

```lua
local COW = `A_C_Cow_01`
RequestModel(COW)
while not HasModelLoaded(COW) do Wait(0) end
local cow = CreatePed(4, COW, 0.0, 0.0, 0.0, 0.0, true, false)
SetModelAsNoLongerNeeded(COW)
```

Common animal hashes: `A_C_Deer_01`, `A_C_Wolf_01`, `A_C_Coyote_01`, `A_C_Bear_01`, `A_C_Cougar_01`, `A_C_Rabbit_01`, `A_C_ChickenHawk_01`.

### Relationship Groups (Law, Enemy, Neutral)

```lua
-- Create groups (once, at boot)
AddRelationshipGroup("MYSCRIPT_NEUTRAL")
AddRelationshipGroup("MYSCRIPT_HOSTILE")
SetRelationshipBetweenGroups(5, `MYSCRIPT_HOSTILE`, `PLAYER`)  -- 5 = hate
SetRelationshipBetweenGroups(5, `PLAYER`, `MYSCRIPT_HOSTILE`)
SetRelationshipBetweenGroups(1, `MYSCRIPT_NEUTRAL`, `PLAYER`)  -- 1 = respect

-- Assign to a ped
SetPedRelationshipGroupHash(ped, `MYSCRIPT_HOSTILE`)
```

## See Also

- [peds/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/index.md)
- [peds/api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/api.md) — full native list: spawn, combat, armour, health, damage, relationship, clothing, props
- [peds/examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/examples.md) — vendor NPC with speech, bandit ambush group, spawn a docile herd of deer
