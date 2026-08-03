---
title: Ped Examples
category: Natives
tags:
  - peds
  - examples
  - ai
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://github.com/femga/rdr3_discoveries
---

# Ped Examples

## Example 1 — Vendor NPC with Ambient Speech + Hat Prop

```lua
local vendorPed = 0
local SHOPKEEP = `S_M_M_VALSHOPKEEPER_01`
local POS = vector4(-308.0, 770.0, 118.5, 180.0)

local function spawnVendor()
    RequestModel(SHOPKEEP)
    while not HasModelLoaded(SHOPKEEP) do Wait(0) end
    vendorPed = CreatePed(26, SHOPKEEP, POS.x, POS.y, POS.z - 1.0, POS.w, false, false)
    SetEntityAsMissionEntity(vendorPed, true, false)
    FreezeEntityPosition(vendorPed, true)
    SetEntityInvincible(vendorPed, true)
    SetBlockingOfNonTemporaryEvents(vendorPed, true)
    SetPedCanBeTargetted(vendorPed, false)
    SetPedCanBeDraggedByAnyVehicle(vendorPed, false)
    SetPedFleeAttributes(vendorPed, 0, 0)
    SetPedCombatAttributes(vendorPed, 0, false)
    SetPedKeepTask(vendorPed, true)

    -- Equip hat (prop 0) for this model index + texture
    SetPedPropIndex(vendorPed, 0, 1, 0, true)

    -- Scenario: sweeping floor
    TaskStartScenarioInPlace(vendorPed, `WORLD_HUMAN_CLEAN`, -1, true, false, false, false)

    SetModelAsNoLongerNeeded(SHOPKEEP)
end

CreateThread(spawnVendor)

-- Periodic ambient speech
CreateThread(function()
    local LINES = { "GREETINGS_PARTNER", "HELLO_GENERIC", "TAKE_YOUR_TIME", "GOODBYE_GENERIC" }
    while true do
        Wait(22000)
        if DoesEntityExist(vendorPed) and not IsPedDeadOrDying(vendorPed, true) then
            local line = LINES[math.random(#LINES)]
            PlayPedAmbientSpeechNative(vendorPed, line, "S_M_M_VALSHOPKEEPER_01_WHISKEY",
                `SPEECH_PARAMS_FORCE_NORMAL_CLEAR`, 0, 0)
        end
    end
end)

AddEventHandler("onResourceStop", function(r)
    if r == GetCurrentResourceName() and DoesEntityExist(vendorPed) then
        SetEntityAsMissionEntity(vendorPed, false, true)
        DeletePed(vendorPed)
        SetEntityAsNoLongerNeeded(vendorPed)
    end
end)
```

---

## Example 2 — Hostile Bandit Ambush (Relationship Group)

```lua
AddRelationshipGroup("BANDITS")
SetRelationshipBetweenGroups(5, `BANDITS`, `PLAYER`)
SetRelationshipBetweenGroups(5, `PLAYER`, `BANDITS`)

local SPAWNS = {
    vector4(1320.0, 200.0, 80.0, 180.0),
    vector4(1325.0, 205.0, 80.0, 160.0),
    vector4(1315.0, 205.0, 80.0, 200.0),
}

local bandits = {}

local function spawnBandit(pos)
    local M = `G_M_M_UniOwanDell_01`
    RequestModel(M)
    while not HasModelLoaded(M) do Wait(0) end
    local ped = CreatePed(29, M, pos.x, pos.y, pos.z, pos.w, true, false)
    SetEntityAsMissionEntity(ped, true, false)
    SetPedRelationshipGroupHash(ped, `BANDITS`)
    SetPedCombatAttributes(ped, 14, true)    -- AlwaysFight
    SetPedCombatAttributes(ped, 2, true)     -- can use ranged
    SetPedCombatAttributes(ped, 5, true)     -- can throw
    SetPedCanBeDraggedByAnyVehicle(ped, false)
    SetPedFleeAttributes(ped, 0, 0)

    -- Give a Cattleman with 24 rounds
    GiveDelayedWeaponToPed(ped, `WEAPON_REVOLVER_CATTLEMAN`, 24, false, true)

    -- Immediately attack player
    TaskCombatPed(ped, PlayerPedId(), 0, 16)
    SetPedKeepTask(ped, true)

    SetModelAsNoLongerNeeded(M)
    table.insert(bandits, ped)
    return ped
end

RegisterCommand("ambush", function()
    for _, s in ipairs(SPAWNS) do spawnBandit(s) end
end)

AddEventHandler("onResourceStop", function(r)
    if r ~= GetCurrentResourceName() then return end
    for _, b in ipairs(bandits) do
        if DoesEntityExist(b) then
            SetEntityAsMissionEntity(b, false, true)
            DeletePed(b)
            SetEntityAsNoLongerNeeded(b)
        end
    end
end)
```

---

## Example 3 — Docile Deer Herd (Flee on Gunshot)

```lua
local DEER = `A_C_Deer_01`
AddRelationshipGroup("DEER_HERD")
SetRelationshipBetweenGroups(3, `DEER_HERD`, `PLAYER`)  -- Fear = flee

local herd = {}
local function spawnDeerAt(pos)
    RequestModel(DEER)
    while not HasModelLoaded(DEER) do Wait(0) end
    local d = CreatePed(4, DEER, pos.x, pos.y, pos.z, 0.0, true, false)
    SetEntityAsMissionEntity(d, true, false)
    SetPedRelationshipGroupHash(d, `DEER_HERD`)
    SetPedFleeAttributes(d, 2, true)               -- flee on gunshot
    SetPedCombatAttributes(d, 14, false)            -- not alwaysFight
    TaskStartScenarioInPlace(d, `WORLD_DEER_GRAZE`, -1, true, false, false, false)
    SetPedKeepTask(d, true)
    SetModelAsNoLongerNeeded(DEER)
    table.insert(herd, d)
    return d
end

-- 5 deer in a small circle
for i = 1, 5 do
    local a = (i / 5) * math.pi * 2
    spawnDeerAt(vector3(-1400.0 + math.cos(a) * 4, -400.0 + math.sin(a) * 4, 65.0))
end

-- If any deer has HP < max, the whole herd flees to nearest forest (simple)
CreateThread(function()
    while true do
        Wait(3000)
        local scared = false
        for _, d in ipairs(herd) do
            if DoesEntityExist(d) and GetEntityHealth(d) < GetPedMaxHealth(d) then
                scared = true; break
            end
        end
        if scared then
            local away = GetEntityCoords(PlayerPedId()) + vector3(200.0, 0.0, 0.0)
            for _, d in ipairs(herd) do
                if DoesEntityExist(d) then
                    ClearPedTasksImmediately(d)
                    TaskWanderInArea(d, away.x, away.y, away.z, 100.0, 5, 50.0, 1)
                end
            end
            scared = false
        end
    end
end)
```

---

## Example 4 — Deputy on Duty with Hat + Sheriff Star

```lua
local DEPUTY = `S_M_M_SheriffDeputy_01`

RequestModel(DEPUTY)
while not HasModelLoaded(DEPUTY) do Wait(0) end

local d = CreatePed(28, DEPUTY, -275.0, 808.0, 119.5 - 1.0, 90.0, true, false)
SetEntityAsMissionEntity(d, true, false)
SetPedRelationshipGroupHash(d, `LAW`)
SetBlockingOfNonTemporaryEvents(d, true)
SetPedKeepTask(d, true)
SetEntityInvincible(d, true)
FreezeEntityPosition(d, true)

-- Sheriff hat + badge
SetPedPropIndex(d, 0, 1, 0, true)     -- Hat component drawable/texture
SetPedPropIndex(d, 10, 0, 0, true)    -- Badge

-- Vest/torso variation (uniform)
SetPedComponentVariation(d, 3, 2, 0, 0)  -- Torso
SetPedComponentVariation(d, 4, 1, 0, 0)  -- Legs
SetPedComponentVariation(d, 6, 3, 0, 0)  -- Boots

-- Give revolver + rifle
GiveDelayedWeaponToPed(d, `WEAPON_REVOLVER_CATTLEMAN`, 50, false, false)
GiveDelayedWeaponToPed(d, `WEAPON_RIFLE_VARMINT`, 60, false, false)

-- Scenario: leaning against post (idle lawman)
TaskStartScenarioInPlace(d, `WORLD_HUMAN_LEANING`, -1, true, false, false, false)

SetModelAsNoLongerNeeded(DEPUTY)
```

---

## Example 5 — Pet Dog (Stay / Follow / Attack)

```lua
local DOG = `A_C_DogHusky_01`

RequestModel(DOG)
while not HasModelLoaded(DOG) do Wait(0) end
local p = PlayerPedId()
local dog = CreatePed(4, DOG, 0, 0, 0, 0, true, false)
SetEntityAsMissionEntity(dog, true, false)

-- Teleport dog to player side
local pos = GetEntityCoords(p) + GetEntityForwardVector(p) * 2.0
SetEntityCoordsNoOffset(dog, pos.x, pos.y, pos.z, false, false, false)

-- Behaviour
SetPedRelationshipGroupHash(dog, `PLAYER`)
SetBlockingOfNonTemporaryEvents(dog, true)
SetPedCanBeTargetted(dog, false)
SetPedKeepTask(dog, true)
SetPedFleeAttributes(dog, 0, 0)
SetPedCombatAttributes(dog, 2, false)
SetPedCombatAttributes(dog, 4, true)    -- melee
SetPedCombatAttributes(dog, 14, true)   -- AlwaysFight for its master

SetModelAsNoLongerNeeded(DOG)

-- Commands
RegisterCommand("dogfollow", function()
    ClearPedTasksImmediately(dog)
    TaskFollowToOffsetOfEntity(dog, p, 2.0, 0.0, 0.0, 2.0, -1, 1.0, 1, 1)
end)
RegisterCommand("dogstay", function()
    ClearPedTasksImmediately(dog)
    TaskStandStill(dog, -1)
end)
RegisterCommand("dogattack", function()
    -- attack nearest non-player ped
    local closest, best = nil, 50.0
    local pp = GetEntityCoords(p)
    for _, id in ipairs(GetActivePlayers()) do
        local tp = GetPlayerPed(id)
        if tp ~= p then
            local d = #(pp - GetEntityCoords(tp))
            if d < best then best, closest = d, tp end
        end
    end
    if closest then
        ClearPedTasksImmediately(dog)
        TaskCombatPed(dog, closest, 0, 16)
    end
end)
```
