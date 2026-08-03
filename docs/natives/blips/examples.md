---
title: Blip Examples
category: Natives
tags:
  - blips
  - examples
  - map
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://github.com/femga/rdr3_discoveries/blob/master/useful_info_from_rpfs/blip_modifiers/README.md
---

# Blip Examples

## Example 1 — Array of Named POIs (Resource-Wide Cleanup)

```lua
local BLIPS = {}

local POIS = {
    { style = `BLIP_STYLE_SHOP`,          pos = vector3(-308.0, 770.0, 118.5), mods = { `BLIP_MODIFIER_MP_COLOR_2` }, name = "General Store" },
    { style = `BLIP_STYLE_AMBIENT_SHERIFF_STAR`, pos = vector3(-278.0, 805.0, 119.5), mods = { `BLIP_MODIFIER_MP_COLOR_8` }, name = "Sheriff Office" },
    { style = `BLIP_STYLE_STAGECOACH`,    pos = vector3(-179.2, 626.3, 114.0), mods = { `BLIP_MODIFIER_MP_COLOR_10` }, name = "Stagecoach" },
    { style = `BLIP_STYLE_MAIL`,          pos = vector3(-175.4, 630.7, 114.0), mods = { `BLIP_MODIFIER_MP_COLOR_15` }, name = "Post Office" },
    { style = `BLIP_STYLE_DINER`,         pos = vector3(-312.3, 816.5, 118.6), mods = { `BLIP_MODIFIER_MP_COLOR_22`, `BLIP_MODIFIER_TOD_DAYTIME_ONLY` }, name = "Saloon" },
}

local function MakeBlip(p)
    local b = BlipAddForCoords(p.style, p.pos.x, p.pos.y, p.pos.z)
    for _, m in ipairs(p.mods) do BlipAddModifier(b, m) end
    SetBlipName(b, p.name)
    SetBlipScale(b, 0.85)
    return b
end

for _, p in ipairs(POIS) do table.insert(BLIPS, MakeBlip(p)) end

AddEventHandler("onResourceStop", function(res)
    if res ~= GetCurrentResourceName() then return end
    for _, b in ipairs(BLIPS) do
        if DoesBlipExist(b) then RemoveBlip(b) end
    end
end)
```

---

## Example 2 — Entity-Tracked Blip for a Horse Stable NPC

```lua
local stablePed = 0  -- a ped you spawned previously
local stableBlip = 0

-- later, once ped is valid:
stableBlip = BlipAddForEntity(`BLIP_STYLE_HORSE`, stablePed)
BlipAddModifier(stableBlip, `BLIP_MODIFIER_MP_COLOR_4`)
SetBlipName(stableBlip, "MacFarlane's Stable Hand")
SetBlipScale(stableBlip, 0.8)

-- auto-remove if the entity is ever destroyed:
CreateThread(function()
    while DoesEntityExist(stablePed) do Wait(1000) end
    if DoesBlipExist(stableBlip) then RemoveBlip(stableBlip) end
end)
```

---

## Example 3 — Pulsing Radius "Bounty Search Area"

```lua
local bountyCenter = vector3(1234.5, 678.9, 78.0)
local bountyRadius = 80.0

local areaStyle  = `BLIP_STYLE_HIDE_AND_SEEK_AREA`
local centerBlip = BlipAddForCoords(`BLIP_STYLE_BOUNTY`, bountyCenter.x, bountyCenter.y, bountyCenter.z)
BlipAddModifier(centerBlip, `BLIP_MODIFIER_ATTENTION`)
SetBlipName(centerBlip, "Bounty: Known Location")

local radiusBlip = BlipAddForRadius(bountyCenter.x, bountyCenter.y, bountyCenter.z, bountyRadius)
BlipAddModifier(radiusBlip, `BLIP_MODIFIER_AREA`)
BlipAddModifier(radiusBlip, `BLIP_MODIFIER_AREA_PULSE`)
BlipAddModifier(radiusBlip, `BLIP_MODIFIER_MP_COLOR_1`)
SetBlipName(radiusBlip, "Search Radius")
```

---

## Example 4 — Blip That Only Shows When Player Is Far Away

Combine two modifiers for a "hide on inside" behaviour:

```lua
local farmCenter = vector3(-1300.0, -300.0, 60.0)
local b = BlipAddForRadius(farmCenter.x, farmCenter.y, farmCenter.z, 40.0)
BlipAddModifier(b, `BLIP_MODIFIER_AREA`)
BlipAddModifier(b, `BLIP_MODIFIER_MP_COLOR_3`)
BlipAddModifier(b, `BLIP_MODIFIER_AREA_HIDE_ON_INSIDE`)
SetBlipName(b, "Downes Ranch")
```

---

## Example 5 — Server-Spawned Blip (via netId + StateBag)

If a server script wants to show *all* clients a blip at a crime scene, broadcast netId of a dummy object + attach a blip client-side:

```lua
-- server
local prop = CreateObjectNoOffset(`p_cs_crate01x`, x, y, z, true, true, false)
local net = NetworkGetNetworkIdFromEntity(prop)
TriggerClientEvent("crimeScene:add", -1, net)

-- client
RegisterNetEvent("crimeScene:add", function(netId)
    while not NetworkDoesNetworkIdExist(netId) do Wait(0) end
    local obj = NetToObj(netId)
    local b = BlipAddForEntity(`BLIP_STYLE_OBJECTIVE`, obj)
    BlipAddModifier(b, `BLIP_MODIFIER_AREA_PULSE`)
    BlipAddModifier(b, `BLIP_MODIFIER_MP_COLOR_1`)
    SetBlipName(b, "Crime Scene")
    SetBlipScale(b, 1.0)
    SetTimeout(60_000, function()
        if DoesBlipExist(b) then RemoveBlip(b) end
    end)
end)
```
