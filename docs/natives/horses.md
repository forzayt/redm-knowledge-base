---
title: Horses
category: Natives
tags:
  - horses
  - mounts
  - redm
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://natives.lawless-street.fr/
  - https://github.com/femga/rdr3_discoveries
  - https://github.com/nativewrappers/rdr3-natives
  - https://alloc8or.re/rdr3/nativedb/?n=TaskMountAnimal
---

# Horses

Horses are **animals (`PED`) with mount-specific natives on top.** They spawn via `CreatePed(pedType=4, model, ...)` and then respond to a family of `*MountAnimal*`, `*Horse*`, and saddle/tack natives.

## Spawn a Horse (Minimal)

```lua
local HORSE = `A_C_Horse_AmericanPaint_Greyovero`
RequestModel(HORSE)
while not HasModelLoaded(HORSE) do Wait(0) end

local horse = CreatePed(4, HORSE, 0.0, 0.0, 0.0, 0.0, true, false)
SetModelAsNoLongerNeeded(HORSE)

SetEntityInvincible(horse, false)
SetEntityAsMissionEntity(horse, true, false)

-- Common tuning
SetHorseReactionLimit(horse, 1.0)           -- 0.0 (skittish) .. 2.0 (steady)
SetHorseCanBeCalmedByPlayer(horse, true)
SetPedCanBeTargetted(horse, false)
SetPedFleeAttributes(horse, 0, 0)
```

### Mount / Dismount

```lua
-- Player mounts the horse (slot -1 = driver)
TaskMountAnimal(PlayerPedId(), horse, 1000, -1, 1.0, 1, 0, 0)

-- Dismount player
TaskDismountAnimal(PlayerPedId(), horse, 0, 0, 0, 0)
```

### Horse Stamina / Bonding

```lua
local stamina = GetHorseStamina(horse)        -- 0.0 .. 1.0
local bonding = GetHorseBondingLevel(horse)   -- 1 .. 4

SetHorseStamina(horse, 1.0)
SetHorseBondingLevel(horse, 4)                -- max bond
```

### Saddles, Stirrups, Blankets

RDR2 attaches tack via the "component" system on horses. Use `SET_PED_COMPONENT_VARIATION` or the higher-level saddle natives:

```lua
-- Saddle id + blanket + stirrups as integer indexes (varies per horse model / content pack)
local SADDLE   = 16   -- example: McCaffrey
local BLANKET  = 3
local STIRRUPS = 4

Citizen.InvokeNative(0xD3A7B003ED343FD9, horse, SADDLE, 1, 1, 1) -- _SET_PED_SADDLE
Citizen.InvokeNative(0x76E4D663EF895195, horse, BLANKET)        -- blanket index
Citizen.InvokeNative(0xE41852E5333B918A, horse, STIRRUPS)       -- stirrups index
```

If a native doesn't have a registered alias yet, use the `Citizen.InvokeNative(0xHASH, args...)` form.

### Follow / Stay / Whistle

```lua
-- Stay in place
TaskStandStill(horse, -1)

-- Follow player (medium distance, unmounted)
TaskFollowToOffsetOfEntity(horse, PlayerPedId(), 2.0, 0.0, 0.0, 1.0, -1, 1.0, 1, 1)

-- Player whistles → horse comes (call native, then TaskGoToEntity)
Citizen.InvokeNative(0x8744EDC06F567909, PlayerPedId(), horse)   -- WHISTLE_FOR_HORSE
TaskGoToEntity(horse, PlayerPedId(), -1, 1.0, 1.0, 0, 0)
```

---

## Popular Horse Models

| Hash literal | Breed / color |
|---|---|
| `A_C_Horse_AmericanPaint_Greyovero` | Paint (Grey Overo) |
| `A_C_Horse_Arabian_White` | Arabian (White) — rare |
| `A_C_Horse_Mustang_GrulloDun` | Mustang (Grullo Dun) |
| `A_C_Horse_Thoroughbred_BlackChestnut` | Thoroughbred (Black Chestnut) |
| `A_C_Horse_KentuckySaddle_ButterMilkBuckskinPainted` | Kentucky Saddle (Buttermilk Buckskin Painted) |
| `A_C_Horse_MissouriFoxtrotter_AmberChampagne` | Missouri Foxtrotter (Amber Champagne) |
| `A_C_Horse_Turkoman_Gold` | Turkoman (Gold) |
| `A_C_Horse_Andalusian_Perlino` | Andalusian (Perlino) |
| `A_C_Donkey_01` | Donkey |
| `A_C_Mule_01` | Mule |

---

## See Also

- [peds/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/index.md) for `CreatePed` + relationship group patterns
- [tasks/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/tasks/index.md) for `TaskGoToEntity`, `TaskFollowToOffsetOfEntity`
- [streaming/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/streaming/index.md) for request/release model patterns
