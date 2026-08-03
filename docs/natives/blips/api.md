---
title: Blips API
category: Natives
tags:
  - blips
  - api
  - map
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://natives.lawless-street.fr/
---

# Blips API

Every native below is listed with Lua alias (if known), hash, namespace, signature, and short notes. Use the alias form when CFX has registered it; otherwise use `Citizen.InvokeNative(0xHASH, ...)`.

Return type `Blip` is just an integer handle (fwScriptGuid index).

---

## Create

### BlipAddForCoords
```
Blip BLIP_ADD_FOR_COORDS(Hash styleHash, float x, float y, float z);
// 0x554D9D53F696D002
// ns HUD
```
Creates a static coordinate marker. This is the 95% use case. See [blips.md](file:///d:/Playground/redm-knowledge-base/docs/natives/blips.md) for style hashes.

### BlipAddForEntity
```
Blip BLIP_ADD_FOR_ENTITY(Hash styleHash, Entity entity);
// 0x12DA2EF8
// ns HUD
```
Attaches blip to entity. Entity movements update blip position automatically (no per-frame `SetBlipCoords` needed).

### BlipAddForRadius
```
Blip BLIP_ADD_FOR_RADIUS(float x, float y, float z, float radius);
// 0x4626756C
// ns HUD
```
Draws a circle of `radius` metres. Works best with styles `BLIP_STYLE_HIDE_AND_SEEK_AREA`, `BLIP_STYLE_MISSION_AREA`, then modifier `BLIP_MODIFIER_AREA` or `AREA_PULSE`.

### BlipAddForArea
```
Blip _ADD_BLIP_FOR_AREA(float x, float y, float z, float width, float height);
// 0x6228F159
// ns HUD (CFX alias: AddBlipForArea)
```
Axis-aligned rectangle blip. Also accepts the same area modifiers.

### BlipAddForPickup
```
Blip BLIP_ADD_FOR_PICKUP(Hash styleHash, Pickup pickup);
// 0x2F9B2F37
// ns HUD
```

---

## Delete

### RemoveBlip
```
void REMOVE_BLIP(Blip* blip);
// 0x86A652570E5F25DD
// ns HUD
```
Takes a **pointer** to your blip variable. Idiom:

```lua
local b = BlipAddForCoords(...)
RemoveBlip(b)   -- Lua passes the number by reference via Lua binding
```

---

## Modifiers

### BlipAddModifier
```
void BLIP_ADD_MODIFIER(Blip blip, Hash modifierHash);
// 0x662D364ABF16DE2F
// ns HUD
```
Apply a modifier (order matters). Apply modifiers BEFORE overriding sprite/text.

### BlipRemoveModifier
```
void BLIP_REMOVE_MODIFIER(Blip blip, Hash modifierHash);
// 0x6FD731CF5F69DEDE
// ns HUD
```

---

## Style / Sprite

### SetBlipSprite
```
void SET_BLIP_SPRITE(Blip blip, Hash spriteHash, int p7);
// 0x21B8968CCF7F1DFB
// ns HUD
```
Override the icon sprite hash. `p7` is usually `0`; pass non-zero for the alternate "radar only" variants. Common sprites: `BLIP_GOLD`, `BLIP_QUEST_A`, `BLIP_AMBIENT_CORPSE`, `BLIP_RADAR_EDGE_ARROW`.

### SetBlipName / SetBlipNameFromPlayerString
```
void SET_BLIP_NAME_FROM_PLAYER_STRING(Blip blip, char* name);   // 0x9CB1A1623062F402
```
Overrides the blip label used in the map legend. **Always call after modifiers.** Use:

```lua
BlipAddModifier(b, `BLIP_MODIFIER_MP_COLOR_1`)
SetBlipName(b, "Valentine Gunsmith")   -- Lua alias
-- or explicitly:
Citizen.InvokeNative(0x9CB1A1623062F402, b, "Valentine Gunsmith")
```

`SetBlipName` is a CFX convenience alias; the canonical native above is what actually runs.

### SetBlipCoords
```
void SET_BLIP_COORDS(Blip blip, float x, float y, float z);
// 0xAE2AF67E9D9AF65D
// ns HUD
```
Move a coordinate blip to a new position. **Not** required for entity-attached blips (they auto-track).

### SetBlipScale
```
void SET_BLIP_SCALE(Blip blip, float scale);
// 0xD3873A5CCBCFB8A4
// ns HUD
```
Visual size multiplier. 1.0 = default. 0.8 = smaller POI, 1.2 = mission objective.

### SetBlipDisplay
```
void SET_BLIP_DISPLAY(Blip blip, int display);
// 0x9029B2F3DA924928
// ns HUD
```
Visibility profile:

| `display` | Meaning |
|---|---|
| 0 | Never show |
| 1 | Mini-map only |
| 2 | Main map only |
| 3 (default) | Both mini-map and main map |
| 8 | Map + selectable |

### SetBlipFade
```
void SET_BLIP_FADE(Blip blip, int fadeTimeMs, int opacity255, BOOL p3);
// 0x3F41EA45DF79ABF6
// ns HUD
```
Smoothly fade a blip in/out over `fadeTimeMs`. Opacity 0 = invisible, 255 = fully opaque.

---

## State / Helpers

### DoesBlipExist
```
BOOL DOES_BLIP_EXIST(Blip blip);
// 0x6102E0C0BA7EA70A
// ns HUD
```
Check validity before `SetBlip*` / `RemoveBlip` when blip handle may come from another client/server message.

### GetBlipCoords
```
Vector3 GET_BLIP_COORDS(Blip blip);
// 0xB6FD1D321D1B264C
// ns HUD
```
Returns the current x, y, z of a coordinate or entity blip.

### GetBlipFromEntity
```
Blip GET_BLIP_FROM_ENTITY(Entity entity);
// 0x5DA94D77
// ns HUD
```
Finds the first active blip attached to an entity, or `0` if none.

### SetBlipHighDetail
```
void SET_BLIP_HIGH_DETAIL(Blip blip, BOOL toggle);
// 0x0BAD253CC444E701
// ns HUD
```
Keeps the sprite rendered at high zoom levels instead of culling early. Use for POIs you want visible from 2 zoom levels out.

### SetBlipCategory
```
void SET_BLIP_CATEGORY(Blip blip, int category);
// 0x1F37E11A
// ns HUD
```
Groups blips under a legend category in the map legend (e.g. 7 = "Stores", 10 = "Jobs"). Usually auto-set by style; modifiers reset it.
