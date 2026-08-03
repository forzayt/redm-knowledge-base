---
title: Peds API
category: Natives
tags:
  - peds
  - api
  - models
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://natives.lawless-street.fr/
---

# Peds API

Signatures + notes for ped-specific natives. Shared entity natives (`GetEntityCoords`, `FreezeEntityPosition`, …) live in [entities/api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/entities/api.md).

---

## Spawn / Delete

### CreatePed
```
Ped CREATE_PED(int pedType, Hash modelHash, float x, float y, float z, float heading, BOOL isNetwork, BOOL bScriptHostPed);
// 0x389EF71 (0xD49F9B0955C367DE)
// ns PED
```
- `pedType`: see [peds/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/peds/index.md). Use `4` for animals, `26` for human male.
- `isNetwork`: network-replicate the ped to other clients. **Use false for local-only vendor NPCs; true for persistent world NPCs.**
- `bScriptHostPed`: pins ownership to the script-host client/server. Usually `false`.

### DeletePed
```
void DELETE_PED(Ped* ped);
// 0x96145A2843EEF7B4
// ns PED
```

### DoesEntityExist / IsPedDeadOrDying
```
BOOL DOES_ENTITY_EXIST(Entity e);              // 0x79911642455C6D4C
BOOL IS_PED_DEAD_OR_DYING(Ped ped, BOOL p1);   // 0x7E26CAF8B046C319
// ns PED / ENTITY
```

---

## Health / Armour / Invincibility

```
int  GET_ENTITY_HEALTH(Entity e);                                       // 0x6176519EBB28C448
void SET_ENTITY_HEALTH(Entity e, int health);                            // 0x607C7A2389B05A08
int  GET_PED_MAX_HEALTH(Ped p);                                          // 0xA2D1B66B46DFC8A3
void SET_PED_MAX_HEALTH(Ped p, int max);                                 // 0x0D92BB4EF091509D
int  GET_PED_ARMOUR(Ped p);                                               // 0xCEB0BBD22A8C6268
void SET_PED_ARMOUR(Ped p, int amount);                                  // 0x09212A0D46F9ED95
void SET_ENTITY_INVINCIBLE(Entity e, BOOL flag);                         // 0x600C1212CE3F1E6F
void SET_PED_SUFFERS_CRITICAL_HITS(Ped p, BOOL toggle);                  // 0xCACCF1082A728CEC
void APPLY_DAMAGE_TO_PED(Ped p, int damage, BOOL p2);                    // 0x6B0541696F9E0CA6
void RESURRECT_PED(Ped p);                                                // 0x77B350BC0396C057
```

---

## Combat / Flee / Behaviour Flags

### SetPedFleeAttributes
```
void SET_PED_FLEE_ATTRIBUTES(Ped ped, int attributes, BOOL toggle);
// 0x8B51B03AF9849738
// ns PED
```
`attributes = 0` disables fleeing entirely (vendor NPC pattern).

### SetPedCombatAttributes
```
void SET_PED_COMBAT_ATTRIBUTES(Ped ped, int attributeIndex, BOOL value);
// 0x3F291D3B89A32162
// ns PED
```
Common attribute indices:

| Index | Name | True meaning |
|---|---|---|
| 0 | BF_CanUseCover | Ped can take cover during combat |
| 2 | BF_CanUseRangedWeapons | Ped can equip/use firearms |
| 4 | BF_CanUseMeleeWeapons | Ped can equip/use melee |
| 5 | BF_CanUseThrownWeapons | Knives, dynamite |
| 14 | BF_AlwaysFight | Never flee, even at 0 HP |
| 17 | BF_CanBeDraggedOutOfVehicle | Prevent this for vendors |
| 46 | BF_CanFightArmedPedsWhenNotArmed | Unarmed will still charge |
| 52 | BF_ForceInjuredOnGround | Force downed behaviour at low HP |
| 58 | BF_CanUsePeekingAroundCover | |
| 78 | BF_DisablePedCanBeUsedAsCover | Prevents nearby ped body shielding |
| 1138 | BF_DisableAutoVictim | Never auto-flee at gunshots |

### SetRelationshipBetweenGroups
```
void SET_RELATIONSHIP_BETWEEN_GROUPS(int relationship, Hash group1, Hash group2);
// 0x45947A51E0D44E62
// ns PED
```
`relationship` values: `0 = Respect`, `1 = Like`, `2 = Ignore`, `3 = Fear`, `4 = Dislike`, `5 = Hate`.

### AddRelationshipGroup
```
int ADD_RELATIONSHIP_GROUP(char* name, Hash* outHash);   // Lua: returns outHash directly
// 0x0E3C4C2CDAD7E37B
// ns PED
```

### SetPedRelationshipGroupHash
```
void SET_PED_RELATIONSHIP_GROUP_HASH(Ped p, Hash groupHash);
// 0x1E85D55581214B6B
// ns PED
```

### SetBlockingOfNonTemporaryEvents
```
void SET_BLOCKING_OF_NON_TEMPORARY_EVENTS(Ped p, BOOL toggle);
// 0xA0FBBB7D44BCE5BA
// ns PED
```
Critical for vendor NPCs. `true` = ped ignores ambient panic, gunshots, and death events around it.

### SetPedKeepTask
```
void SET_PED_KEEP_TASK(Ped p, BOOL keep);
// 0xFE2819E665E48F3B
// ns PED
```
`true` disables idle AI task overwrites (walking off, etc.). Use with `TaskStartScenarioInPlace` so NPCs don't wander.

### SetPedCanBeTargetted
```
void SET_PED_CAN_BE_TARGETTED(Ped p, BOOL targetable);
// 0x36A62F295371AB5C
// ns PED
```
`false` removes the ped from free-aim assist. Good for vendors + barkeepers.

### SetPedCanBeDraggedByAnyVehicle
```
void SET_PED_CAN_BE_DRAGGED_BY_ANY_VEHICLE(Ped p, BOOL can);
// 0x2FCABF48BB30A244
// ns PED
```
`false` prevents wagons from flinging the ped across the map on collision.

---

## Props / Clothing Components

RDR2 uses a component system similar to GTA V but with much richer per-model tables.

### SetPedComponentVariation (main components)
```
void SET_PED_COMPONENT_VARIATION(Ped ped, int componentId, int drawable, int texture, int palette);
// 0xD4F7BEB13C1C8B18  (0xE2DF122CDD75971F)
// ns PED
```
Common `componentId` (for human `mp_male`/`mp_female`):

| Id | Slot |
|---|---|
| 0 | Face / head |
| 1 | Mask / beard overlay |
| 2 | Hair |
| 3 | Torso |
| 4 | Legs |
| 5 | Hands / gloves |
| 6 | Feet / boots |
| 7 | Eyes / teeth |
| 8 | Accessories (scarves, neck ties, neckerchiefs) |
| 9 | Undershirt / vest |
| 10 | Decals / badges |
| 11 | Auxiliary overlay (coat overlays) |

### SetPedPropIndex / ClearPedProp
```
void SET_PED_PROP_INDEX(Ped ped, int propId, int drawableIndex, int textureId, BOOL attach);
// 0x0532AC192F925D32
// ns PED

void CLEAR_PED_PROP(Ped ped, int propId);
// 0x2D23D743493C830F
// ns PED
```
Common `propId`:

| Id | Prop |
|---|---|
| 0 | Hat |
| 1 | Glasses |
| 2 | Earring |
| 6 | Watch |
| 7 | Wristband |
| 9 | Belt buckles / accessories |
| 10 | Badge |
| 11 | Cigar / cigarette (mouth props) |

### GetPedDrawableVariation / GetPedTextureVariation / GetNumberOfPedDrawableVariations
```
int GET_PED_DRAWABLE_VARIATION(Ped p, int componentId);           // 0x1389480A9CDAA206
int GET_PED_TEXTURE_VARIATION(Ped p, int componentId);            // 0xF898E33452D6C6F0
int GET_NUMBER_OF_PED_DRAWABLE_VARIATIONS(Ped p, int componentId); // 0x5AEF600298449270
```

### _SetPedSaddle (horse)
See [horses.md](file:///d:/Playground/redm-knowledge-base/docs/natives/horses.md).

---

## Weapons / Combat Equip

```
void GIVE_DELAYED_WEAPON_TO_PED(Ped p, Hash weaponHash, int ammoCount, BOOL isHidden, BOOL bForceInHand);
// 0xB28285312C116C93
// ns PED
```
Prefer this over `GIVE_WEAPON_TO_PED` — "delayed" is the engine's newer path and works reliably on networked peds.

```
void SET_CURRENT_PED_WEAPON(Ped p, Hash weaponHash, BOOL bForceInHand, BOOL bIgnoreAmmo, BOOL bAllowDuplicates, int p5);
// 0x0F07B78F0F6F1BC5
// ns PED
```

```
BOOL REMOVE_WEAPON_FROM_PED(Ped p, Hash weaponHash, BOOL p2);
// 0x7E8DC7E52750DF9A
// ns PED

void REMOVE_ALL_PED_WEAPONS(Ped p, BOOL arg);
// 0xA712B7EF29D0506E
// ns PED
```

---

## Speech / Ambient Lines

```
void PLAY_PED_AMBIENT_SPEECH_WITH_VOICE_NATIVE(Ped p, char* speechName, char* voiceName, Hash speechParamHash, BOOL p4, BOOL p5);
// 0x8E04FEDD28D42462
// ns AUDIO (PED wrapper)
```
The easiest way to make NPCs say lines. `speechName` examples: `TAKE_YOUR_TIME`, `HELLO_GENERIC`, `GOODBYE_GENERIC`, `GENERIC_HOWDY`, `GREETINGS_PARTNER`.

Example:
```lua
PlayPedAmbientSpeechNative(vendorPed, "HELLO_GENERIC", "S_M_M_VALSHOPKEEPER_01_WHISKEY", "SPEECH_PARAMS_FORCE", 0, 0)
```
(Note: `PlayPedAmbientSpeechNative` is a CFX alias for the long native above; use it if available.)

---

## Cold / Heat (RedM Weather)

```
void APPLY_COLD_TO_PED(Ped ped, float intensity, float p2);        // 0xEB9058C99F5C43FA
void APPLY_HEAT_TO_PED(Ped ped, float intensity, float p2);        // 0x0C1D2D4419F47C5E
```

Use `intensity` in the range `0.0` (nothing) to `1.0` (freezing / scorching). Affects shivering animation, sweat effects, and stamina drain.
