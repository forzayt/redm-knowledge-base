---
title: Blips
category: Natives
tags:
  - blips
  - ui
  - map
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://natives.lawless-street.fr/
  - https://github.com/femga/rdr3_discoveries/blob/master/useful_info_from_rpfs/blip_modifiers/README.md
  - https://alloc8or.re/rdr3/nativedb/?n=BlipAddForCoords
---

# Blips

Blips are the HUD/map markers the engine renders on the mini-map, legend, and pause menu. RedM uses **style hashes** (not FiveM-style sprite integers) as the base of every blip, then overlays modifiers like color, category, pulse, and attention on top.

## Types of Blip

| Blip type | Native | Description |
|---|---|---|
| **Coordinate marker** | `BlipAddForCoords(style, x, y, z)` | Fixed point on map (shops, POIs, NPCs). Most common. |
| **Entity marker** | `BlipAddForEntity(style, entity)` | Attaches to a ped/vehicle/object and tracks it. |
| **Radius** | `BlipAddForRadius(x, y, z, radius)` | Search-area circle. Useful for bounty radius / hide-and-seek. |
| **Area (rectangle)** | `BlipAddForArea(x, y, z, w, h)` | Rectangular area. |
| **Pickup (style)** | `BlipAddForPickup(style, pickup)` | Attaches to a pickup (money, ammo, corpse). |

## Style Base + Modifier = Final Look

Every blip starts with a **style hash** chosen from `BLIP_STYLE_*`. Overlay zero or more **modifier hashes** (`BLIP_MODIFIER_*`) to tweak colour, range, overlays, alpha, or conditions (TOD, inside/outside).

Common style hashes:

| Hash | Use |
|---|---|
| `BLIP_STYLE_SHOP` | General stores, stables |
| `BLIP_STYLE_OBJECTIVE` | Main story / mission marker |
| `BLIP_STYLE_PICKUP` | Pickup / collectible (bag of dollars) |
| `BLIP_STYLE_PLAYER` | Generic player/crew (auto-route) |
| `BLIP_STYLE_WAYPOINT` | Custom waypoint marker |
| `BLIP_STYLE_AMBIENT_SHERIFF_STAR` | Law / sheriff office |
| `BLIP_STYLE_AMBULANCE` | Doctor / medic |

Common modifiers:

| Hash | Effect |
|---|---|
| `BLIP_MODIFIER_MP_COLOR_1` … `MP_COLOR_32` | Apply a team / category color |
| `BLIP_MODIFIER_AREA_PULSE` | Smooth alpha pulse on area blips |
| `BLIP_MODIFIER_ATTENTION` | Attention overlay (star/exclamation) in white |
| `BLIP_MODIFIER_TOD_DAYTIME_ONLY` | Only show during daytime |
| `BLIP_MODIFIER_COMPASS_OBJECTIVE` | 35m objective behaviour (hide when close) |
| `BLIP_MODIFIER_AREA_DIRECTIONAL` | Edge arrow + area style |
| `BLIP_MODIFIER_CULL_ON_DEATH` | Hide when host entity dies |
| `BLIP_MODIFIER_ENEMY` | Enemy threat / colour |
| `BLIP_MODIFIER_COMPANION_WOUNDED` | Revive overlay |

---

## Full Example

```lua
local myBlips = {}

local function CreateBlip(style, coords, modifiers, name)
    local blip = BlipAddForCoords(style, coords.x, coords.y, coords.z)
    for _, mod in ipairs(modifiers) do
        BlipAddModifier(blip, mod)
    end
    SetBlipName(blip, name)       -- ALWAYS set name AFTER modifiers
    SetBlipSprite(blip, `BLIP_GOLD`, 0) -- optional: override sprite hash + p7
    table.insert(myBlips, blip)
    return blip
end

CreateBlip(
    `BLIP_STYLE_SHOP`,
    vector3(-308.0, 770.0, 118.5),
    { `BLIP_MODIFIER_MP_COLOR_2`, `BLIP_MODIFIER_TOD_DAYTIME_ONLY` },
    "Valentine General Store"
)

CreateBlip(
    `BLIP_STYLE_AMBIENT_SHERIFF_STAR`,
    vector3(-278.0, 805.0, 119.5),
    { `BLIP_MODIFIER_MP_COLOR_8` },
    "Valentine Sheriff"
)

AddEventHandler("onResourceStop", function(res)
    if res == GetCurrentResourceName() then
        for _, b in ipairs(myBlips) do RemoveBlip(b) end
    end
end)
```

---

## See Also

- [blips/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/blips/index.md)
- [blips/api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/blips/api.md) — complete native signatures + per-native notes
- [blips/examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/blips/examples.md) — recipes for entity blips, radius blips, blip-for-ped
