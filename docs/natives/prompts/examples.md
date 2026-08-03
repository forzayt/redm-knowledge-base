---
title: Prompt Examples
category: Natives
tags:
  - prompts
  - examples
  - interaction
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
---

# Prompt Examples

## Example 1 — Simple E Prompt (Press)

```lua
local SHOP_POS = vector3(-308.0, 770.0, 118.5)
local openShop = nil

-- Register once, ever.
do
    PromptRegisterBegin()
    PromptSetControlAction(0, `INPUT_CONTEXT`)
    PromptSetText(0, CreateVarString(10, "LITERAL_STRING", "Open Shop"))
    PromptSetEnabled(0, false)
    PromptSetVisible(0, false)
    PromptSetStandardMode(0, true)
    openShop = PromptRegisterEnd()
end

-- Distance gating + press check
CreateThread(function()
    while true do
        Wait(0)
        local ped = PlayerPedId()
        local near = #(SHOP_POS - GetEntityCoords(ped)) < 2.0
        PromptSetEnabled(openShop, near)
        PromptSetVisible(openShop, near)
        if near and IsControlJustReleased(0, `INPUT_CONTEXT`) then
            -- TriggerServerEvent("myscript:openShop") ...
            print("Open shop")
        end
    end
end)

AddEventHandler("onResourceStop", function(r)
    if r == GetCurrentResourceName() then PromptDelete(openShop) end
end)
```

---

## Example 2 — Hold 1.5s Before Trigger

Useful for chopping trees, mining, lockpicking.

```lua
local chopTree = nil
do
    PromptRegisterBegin()
    PromptSetControlAction(0, `INPUT_CONTEXT`)
    PromptSetText(0, CreateVarString(10, "LITERAL_STRING", "Chop Tree"))
    PromptSetHoldMode(0, true)
    PromptSetHoldTime(0, 1500)
    PromptSetEnabled(0, false)
    PromptSetVisible(0, false)
    chopTree = PromptRegisterEnd()
end

local TREE_POS = vector3(-1250.0, -200.0, 65.0)

CreateThread(function()
    while true do
        Wait(0)
        local near = #(TREE_POS - GetEntityCoords(PlayerPedId())) < 1.8
        PromptSetEnabled(chopTree, near)
        PromptSetVisible(chopTree, near)
        if near and PromptHasHoldModeCompleted(chopTree) then
            print("1.5s chop complete")
            -- spawn axe anim, then give wood, etc.
        end
    end
end)
```

---

## Example 3 — Grouped Prompt (Corpse: Loot E / Hide R / Examine G)

Grouped prompts share a single anchor point and only display when you call `UiPromptSetActiveGroupThisFrame` that tick.

```lua
local CORPSE_GROUP = 1
local lootPrompt, hidePrompt, examPrompt = nil, nil, nil

local function makePrompt(key, text)
    PromptRegisterBegin()
    PromptSetControlAction(0, key)
    PromptSetText(0, CreateVarString(10, "LITERAL_STRING", text))
    PromptSetStandardMode(0, true)
    PromptSetEnabled(0, false)
    PromptSetVisible(0, false)
    PromptSetGroup(0, CORPSE_GROUP)
    return PromptRegisterEnd()
end

lootPrompt = makePrompt(`INPUT_CONTEXT`,         "Loot")
hidePrompt = makePrompt(`INPUT_RELOAD`,          "Hide Body")
examPrompt = makePrompt(`INPUT_INTERACT_ANIMAL`, "Examine")

CreateThread(function()
    while true do
        Wait(0)
        -- Check nearest corpse within 2.0 m
        local nearCorpse, closestDist = nil, 2.0
        local pp = GetEntityCoords(PlayerPedId())
        -- In production this would iterate through known spawned corpses;
        -- for brevity here assume a fixed test position.
        local testCorpse = vector3(-200.0, 800.0, 119.0)
        local dist = #(testCorpse - pp)
        if dist < closestDist then nearCorpse = true end

        if nearCorpse then
            UiPromptSetActiveGroupThisFrame(CORPSE_GROUP, true)
            PromptSetEnabled(lootPrompt, true)
            PromptSetVisible(lootPrompt, true)
            PromptSetEnabled(hidePrompt, true)
            PromptSetVisible(hidePrompt, true)
            PromptSetEnabled(examPrompt, true)
            PromptSetVisible(examPrompt, true)

            if IsControlJustReleased(0, `INPUT_CONTEXT`)         then print("Loot") end
            if IsControlJustReleased(0, `INPUT_RELOAD`)          then print("Hide Body") end
            if IsControlJustReleased(0, `INPUT_INTERACT_ANIMAL`) then print("Examine") end
        else
            PromptSetEnabled(lootPrompt, false)
            PromptSetVisible(lootPrompt, false)
            PromptSetEnabled(hidePrompt, false)
            PromptSetVisible(hidePrompt, false)
            PromptSetEnabled(examPrompt, false)
            PromptSetVisible(examPrompt, false)
        end
    end
end)

AddEventHandler("onResourceStop", function(r)
    if r ~= GetCurrentResourceName() then return end
    PromptDelete(lootPrompt)
    PromptDelete(hidePrompt)
    PromptDelete(examPrompt)
end)
```

---

## Example 4 — Dynamic Prompt Text (Different Label Per Object)

```lua
local generalPrompt = nil
do
    PromptRegisterBegin()
    PromptSetControlAction(0, `INPUT_CONTEXT`)
    PromptSetEnabled(0, false)
    PromptSetVisible(0, false)
    PromptSetStandardMode(0, true)
    generalPrompt = PromptRegisterEnd()
end

CreateThread(function()
    while true do
        Wait(0)
        local p, nearest, label = GetEntityCoords(PlayerPedId()), nil, nil
        local best = 2.0

        -- Imagine these are scanned via raycast or tracked entity table.
        local scan = {
            { pos = vector3(-250.0, 700.0, 118.0), label = "Pick up Hat" },
            { pos = vector3(-255.0, 705.0, 118.0), label = "Open Chest" },
        }

        for _, o in ipairs(scan) do
            local d = #(o.pos - p)
            if d < best then best, nearest, label = d, o, o.label end
        end

        if nearest then
            PromptSetText(generalPrompt,
                CreateVarString(10, "LITERAL_STRING", label))
            PromptSetEnabled(generalPrompt, true)
            PromptSetVisible(generalPrompt, true)
            if IsControlJustReleased(0, `INPUT_CONTEXT`) then
                print("Interact:", label)
            end
        else
            PromptSetEnabled(generalPrompt, false)
            PromptSetVisible(generalPrompt, false)
        end
    end
end)
```

---

## Example 5 — Disable Default Input + Custom Prompt

Prevents the built-in pickup interaction when E is pressed for a custom interaction.

```lua
local unlockDoor = nil
do
    PromptRegisterBegin()
    PromptSetControlAction(0, `INPUT_CONTEXT`)
    PromptSetText(0, CreateVarString(10, "LITERAL_STRING", "Unlock"))
    PromptSetHoldMode(0, true)
    PromptSetHoldTime(0, 1000)
    PromptSetEnabled(0, false)
    PromptSetVisible(0, false)
    unlockDoor = PromptRegisterEnd()
end

local DOOR = vector3(-278.0, 810.0, 119.5)

CreateThread(function()
    while true do
        Wait(0)
        local near = #(DOOR - GetEntityCoords(PlayerPedId())) < 1.8
        if near then
            DisableControlAction(0, `INPUT_CONTEXT`, true)
            PromptSetEnabled(unlockDoor, true)
            PromptSetVisible(unlockDoor, true)
            if PromptHasHoldModeCompleted(unlockDoor) then
                print("Door unlocked")
            end
        else
            PromptSetEnabled(unlockDoor, false)
            PromptSetVisible(unlockDoor, false)
        end
    end
end)
```
