---
title: Entity Examples
category: Natives
tags:
  - entities
  - examples
  - networking
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
---

# Entity Examples

## Example 1 — Static Prop (Resource-Wide Cleanup)

```lua
local createdObjects = {}

local function spawnProp(model, coords, heading, opts)
    opts = opts or {}
    RequestModel(model)
    while not HasModelLoaded(model) do Wait(0) end

    local obj = CreateObject(model, coords.x, coords.y, coords.z,
        opts.isNetworked or false, false, opts.isDoor or false)

    SetEntityAsMissionEntity(obj, true, false)
    SetEntityHeading(obj, heading or 0.0)
    FreezeEntityPosition(obj, opts.frozen ~= false)
    SetEntityInvincible(obj, opts.invincible ~= false)
    if opts.onGround then PlaceObjectOnGroundProperly(obj) end
    if opts.collision == false then
        SetEntityCollision(obj, false, false)
    end

    SetModelAsNoLongerNeeded(model)
    table.insert(createdObjects, obj)
    return obj
end

-- Valentine spawn: hitching post + 2 barrels
local post = spawnProp(`p_hitchingpost01x`,   vector3(-303.0, 773.0, 118.0), 0.0, {onGround=true})
spawnProp(`prop_barrel_02b`, vector3(-305.0, 773.5, 118.0), 20.0, {onGround=true})
spawnProp(`prop_barrel_02b`, vector3(-305.0, 772.5, 118.0), -45.0, {onGround=true})

AddEventHandler("onResourceStop", function(res)
    if res ~= GetCurrentResourceName() then return end
    for _, o in ipairs(createdObjects) do
        if DoesEntityExist(o) then
            SetEntityAsMissionEntity(o, false, true)
            DeleteObject(o)
            SetEntityAsNoLongerNeeded(o)
        end
    end
end)
```

---

## Example 2 — Carryable Lantern Attached to Player's Belt

```lua
local LANTERN = `p_lantern01x`
local playerLanterns = {}  -- [source] = object handle

local function attachLanternToPed(ped)
    RequestModel(LANTERN)
    while not HasModelLoaded(LANTERN) do Wait(0) end
    local prop = CreateObject(LANTERN, 0, 0, 0, false, false, false)
    local bone = GetEntityBoneIndexByName(ped, "BELT_LEFT")  -- usually 40269
    AttachEntityToEntity(prop, ped, bone,
        -0.15, -0.02, -0.02,      -- x y z offset (cm)
        90.0, 0.0, 0.0,            -- x y z rotation
        false, true, false, false, -- p9, soft pinning, collision, isPed
        2, true)                   -- rotation order, fixed rot
    SetModelAsNoLongerNeeded(LANTERN)
    return prop
end

RegisterCommand("beltlantern", function()
    local ped = PlayerPedId()
    if playerLanterns[1] and DoesEntityExist(playerLanterns[1]) then
        DetachEntity(playerLanterns[1], false, false)
        DeleteObject(playerLanterns[1])
        playerLanterns[1] = nil
    else
        playerLanterns[1] = attachLanternToPed(ped)
    end
end)
```

---

## Example 3 — Wagon + NPC Driver (Stagecoach)

```lua
local WAGON = `STAGECOACH001X`
local DRIVER = `S_M_M_CoachDriver_01`

local function spawnWagonWithDriver(coords, heading)
    -- Wagon
    RequestModel(WAGON)
    while not HasModelLoaded(WAGON) do Wait(0) end
    local wagon = CreateVehicle(WAGON, coords.x, coords.y, coords.z, heading, true, false)
    SetVehicleOnGroundProperly(wagon)
    SetModelAsNoLongerNeeded(WAGON)

    -- Driver (seat -1 = driver)
    RequestModel(DRIVER)
    while not HasModelLoaded(DRIVER) do Wait(0) end
    local driver = CreatePedInsideVehicle(wagon, 26, DRIVER, -1, true, false)
    SetBlockingOfNonTemporaryEvents(driver, true)
    SetModelAsNoLongerNeeded(DRIVER)

    -- Drive to a waypoint
    local dest = vector3(coords.x + 50.0, coords.y, coords.z)
    TaskVehicleDriveToCoordLongrange(driver, wagon, dest.x, dest.y, dest.z,
        8.0, 1574777268, 25.0, 2)  -- cruise speed, driving style, stop-dist, longRange flag

    return wagon, driver
end

local w, d = spawnWagonWithDriver(vector3(-180.0, 640.0, 114.0), 90.0)
```

---

## Example 4 — Locked Saloon Door (Networked)

```lua
local DOOR_MODEL = `p_door_valh02x`
local DOOR_POS   = vector4(-312.3, 816.3, 118.6, 0.0)

RequestModel(DOOR_MODEL)
while not HasModelLoaded(DOOR_MODEL) do Wait(0) end

local door = CreateObject(DOOR_MODEL, DOOR_POS.x, DOOR_POS.y, DOOR_POS.z,
    true, false, true)   -- isDoor = true enables door physics mode
SetEntityAsMissionEntity(door, true, false)
SetEntityHeading(door, DOOR_POS.w)
FreezeEntityPosition(door, false)
SetEntityInvincible(door, true)
SetModelAsNoLongerNeeded(DOOR_MODEL)

-- Freeze = locked; unfreeze to open
local locked = true
local function setDoorLock(state)
    locked = state
    FreezeEntityPosition(door, state)
end

RegisterCommand("toggledoor", function()
    setDoorLock(not locked)
end)
```

---

## Example 5 — Drop Item Prop + Pickup (Client<->Server)

```lua
-- server.lua
RegisterNetEvent("myscript:dropItem", function(item, amount, pos)
    -- Validate count, inventory, etc here...
    local src = source
    -- Spawn a networked barrel-like prop server-side, then broadcast netId
    local MODEL = `mp001_s_mp_prop_box01x`
    local obj = CreateObject(MODEL, pos.x, pos.y, pos.z, true, true, false)
    while not DoesEntityExist(obj) do Wait(0) end
    local netId = NetworkGetNetworkIdFromEntity(obj)
    TriggerClientEvent("myscript:itemDropped", -1, netId, item, amount, src)
end)

-- client.lua
local drops = {}  -- [netId] = {item, amount, source}

RegisterNetEvent("myscript:itemDropped", function(netId, item, amt, src)
    while not NetworkDoesNetworkIdExist(netId) do Wait(0) end
    drops[netId] = {item=item, amount=amt, src=src}
    local obj = NetToObj(netId)
    PlaceObjectOnGroundProperly(obj)
end)

-- Simple pickup prompt (poll if player is near any drop)
CreateThread(function()
    while true do Wait(500)
        local pos = GetEntityCoords(PlayerPedId())
        for netId, drop in pairs(drops) do
            local obj = NetworkDoesNetworkIdExist(netId) and NetToObj(netId) or 0
            if obj ~= 0 and DoesEntityExist(obj) then
                local dist = #(pos - GetEntityCoords(obj))
                if dist < 1.5 then
                    print(("PICKUP: %s x%d"):format(drop.item, drop.amount))
                    -- emit pickup event, server deletes obj via DeleteObject + SetEntityAsNoLongerNeeded
                end
            end
        end
    end
end)
```
