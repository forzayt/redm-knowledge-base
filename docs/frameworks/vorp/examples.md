---
title: VORP Examples
category: Frameworks
tags:
  - vorp
  - examples
  - framework
framework:
  - vorp
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/core
  - https://docs.vorp-core.com/api-reference/lib
---

# VORP Examples

Practical, copy-pasteable patterns that combine the Core, Inventory, Character, and Lib APIs.

---

## Example 1 — Character Lookup + Greeting

Server-side greeting that looks up a character and prints a welcome message.

```lua
-- server/main.lua
local Core = exports.vorp_core:GetCore()

RegisterNetEvent("myscript:requestWelcome", function()
    local source = source

    local user = Core.getUser(source)
    if not user then return end

    local char = user.getUsedCharacter
    if not char then return end

    local fullName = ("%s %s"):format(char.firstname, char.lastname)
    local jobText  = char.jobLabel and ("Job: " .. char.jobLabel) or "Job: Unemployed"

    Core.NotifySimpleTop(source,
        ("Welcome, %s"):format(fullName),
        ("$%d | Gold %d | %s"):format(char.money, char.gold, jobText),
        5000
    )
end)
```

```lua
-- client/main.lua
RegisterNetEvent("vorp:SelectedCharacter", function()
    TriggerServerEvent("myscript:requestWelcome")
end)
```

---

## Example 2 — /pay Command (Give Money to Nearby Player)

Server-side command that finds the nearest player and transfers money.

```lua
-- server/main.lua
local Core = exports.vorp_core:GetCore()

local function getNearestPlayer(source, maxDist)
    local srcPed = GetPlayerPed(source)
    local srcPos = GetEntityCoords(srcPed)
    local closest, closestDist = nil, maxDist or 5.0

    for _, id in ipairs(GetPlayers()) do
        if tonumber(id) ~= source then
            local ped = GetPlayerPed(tonumber(id))
            local pos = GetEntityCoords(ped)
            local dist = #(srcPos - pos)
            if dist < closestDist then
                closest, closestDist = tonumber(id), dist
            end
        end
    end
    return closest
end

RegisterCommand("pay", function(source, args)
    if source == 0 then return end
    local amount = tonumber(args[1] or 0)
    if amount <= 0 then
        Core.NotifyTip(source, "Usage: /pay [amount]", 3000)
        return
    end

    local user = Core.getUser(source)
    if not user then return end
    local char = user.getUsedCharacter
    if not char then return end

    local target = getNearestPlayer(source, 5.0)
    if not target then
        Core.NotifyFail(source, "No player nearby", "Get closer first", 3000)
        return
    end

    if char.money < amount then
        Core.NotifyFail(source, "Not enough money", "You don't have enough cash", 3000)
        return
    end

    local targetUser = Core.getUser(target)
    if not targetUser then return end
    local targetChar = targetUser.getUsedCharacter
    if not targetChar then return end

    char.deductMoney(amount)
    targetChar.addMoney(amount)

    Core.NotifyTip(source, ("Paid $%d to %s %s"):format(amount, targetChar.firstname, targetChar.lastname), 3000)
    Core.NotifyTip(target, ("Received $%d from %s %s"):format(amount, char.firstname, char.lastname), 3000)
end, false)
```

---

## Example 3 — Item Shop: Buy / Sell (Inventory API)

A simple vendor NPC: press E near coords to buy bread or sell raw meat.

```lua
-- shared/config.lua
Config = {}
Config.Shop = {
    name   = "Valentine General",
    coords = vector3(-310.83, 772.80, 118.70),
    heading = 160.0,
    pedModel = "S_M_M_ValShopKeeper_01",
    prices = {
        buy  = { bread = 1, water = 2, bandage = 5 },
        sell = { meat_raw = 2, pelt_deer = 5 }
    }
}
```

```lua
-- server/main.lua
local Core = exports.vorp_core:GetCore()
local INV = exports.vorp_inventory

-- Promise wrapper for inventory callbacks
local function p(fn)
    return function(...)
        local args = {...}
        local result
        local p = promise.new()
        args[#args + 1] = function(r) p:resolve(r) end
        fn(table.unpack(args))
        return Citizen.Await(p)
    end
end
local canCarry   = p(function(...) INV:canCarryItem(...) end)
local addItem    = p(function(...) INV:addItem(...) end)
local subItem    = p(function(...) INV:subItem(...) end)
local getItemCnt = p(function(...) INV:getItemCount(...) end)

RegisterNetEvent("myscript:buy", function(item)
    local source = source
    local price  = Config.Shop.prices.buy[item]
    if not price then return end

    local user = Core.getUser(source)
    if not user then return end
    local char = user.getUsedCharacter

    if char.money < price then
        Core.NotifyFail(source, "Cannot buy", "Not enough money", 3000)
        return
    end
    if not canCarry(source, item, 1) then
        Core.NotifyFail(source, "Cannot buy", "Inventory full", 3000)
        return
    end

    char.deductMoney(price)
    local ok = addItem(source, item, 1, {})
    if ok then
        Core.NotifyTip(source, ("Bought 1x %s for $%d"):format(item, price), 3000)
    end
end)

RegisterNetEvent("myscript:sell", function(item)
    local source = source
    local price  = Config.Shop.prices.sell[item]
    if not price then return end

    local count = getItemCnt(source, item, nil)
    if count <= 0 then
        Core.NotifyFail(source, "Cannot sell", "You have none", 3000)
        return
    end

    local user = Core.getUser(source)
    local char = user.getUsedCharacter

    local ok = subItem(source, item, 1, {}, nil, false, nil)
    if ok then
        char.addMoney(price)
        Core.NotifyTip(source, ("Sold 1x %s for $%d"):format(item, price), 3000)
    end
end)
```

```lua
-- client/main.lua
-- Uses VORP Lib: inputs, prompts, blips, entities
local Lib = Import({"prompts", "blips", "entities"})
local Prompts = Lib.Prompts
local Blips   = Lib.Blips
local Entity  = Lib.entities

local Core = exports.vorp_core:GetCore()

local shopPrompt, shopNpc

-- Init when character is selected
RegisterNetEvent("vorp:SelectedCharacter", function()
    -- Blip
    local b = Blips.Blips:Create("coords", {
        Pos = Config.Shop.coords,
        Blip = 1673015813,
        Options = { name = Config.Shop.name, color = "blue" }
    })

    -- NPC
    shopNpc = Entity.Ped:Create({
        Model = Config.Shop.pedModel,
        Pos = vector4(Config.Shop.coords.x, Config.Shop.coords.y, Config.Shop.coords.z, Config.Shop.heading),
        IsNetworked = false,
        Options = { PlaceOnGround = true },
        OnCreate = function(self)
            FreezeEntityPosition(self:GetHandle(), true)
            SetEntityInvincible(self:GetHandle(), true)
        end
    })

    -- Prompt
    shopPrompt = Prompts.Prompts:Register({
        text = "Open " .. Config.Shop.name,
        key = 0xE832C8BB,   -- E
        group = 0,
        visible = false,
        enabled = false,
        onPressed = function()
            local menu = {
                title    = Config.Shop.name,
                subtext  = "Select an action",
                items = {}
            }
            for item, price in pairs(Config.Shop.prices.buy) do
                table.insert(menu.items, {
                    label = ("Buy %s — $%d"):format(item, price),
                    value = "buy", args = item
                })
            end
            for item, price in pairs(Config.Shop.prices.sell) do
                table.insert(menu.items, {
                    label = ("Sell %s — $%d"):format(item, price),
                    value = "sell", args = item
                })
            end

            -- For this example we just use NotifyLeft interactive-style prompts
            -- In production, use vorp_menu or a custom NUI menu
            for i, it in ipairs(menu.items) do
                Core.NotifyLeft(it.label, "Press " .. (i == 1 and "1" or i == 2 and "2" or "3"),
                    "toast_mp", "icon_proc_loadingcircle", 4000, "white")
            end
        end
    })

    -- Proximity enable loop
    CreateThread(function()
        while true do
            Wait(500)
            local dist = #(Config.Shop.coords - GetEntityCoords(PlayerPedId()))
            local near = dist < 3.0
            shopPrompt:SetEnabled(near)
            shopPrompt:SetVisible(near)
        end
    end)
end)
```

---

## Example 4 — Metadata Item: Custom Document with Context Actions

Create a "document" item that stores arbitrary text and has READ / DESTROY right-click actions.

```lua
-- server/main.lua
local Core = exports.vorp_core:GetCore()
local INV = exports.vorp_inventory

-- 1. Whitelist the context-menu server events FIRST
INV:addAllowedContextMenuEvent({
    "myscript:doc:read",
    "myscript:doc:destroy",
}, GetCurrentResourceName())

-- 2. Command to give a test document
RegisterCommand("givedoc", function(source, args, raw)
    local src = source
    local user = Core.getUser(src)
    if not user then return end
    local char = user.getUsedCharacter

    local text = table.concat(args, " ") or "This is an official document."

    local metadata = {
        label       = "Wanted Poster",
        description = text,
        context     = {
            {
                text  = "READ",
                close = true,
                event = { server = "myscript:doc:read", client = "myscript:doc:read" },
                arguments = { text }
            },
            {
                text  = "TEAR UP",
                close = true,
                event = { server = "myscript:doc:destroy" }
            }
        }
    }

    INV:addItem(src, "document", 1, metadata, function(ok)
        if ok then
            Core.NotifyTip(src, "Document added", 3000)
        else
            Core.NotifyFail(src, "Inventory full", "Clear space first", 3000)
        end
    end)
end, true)

-- 3. Server handlers for context events
AddEventHandler("myscript:doc:read", function(source, args, itemId)
    -- args[1] = text, itemId = the item slot id
    local text = args[1] or ""
    Core.NotifyCenter(source, text, 7000)
end)

AddEventHandler("myscript:doc:destroy", function(source, args, itemId)
    if not itemId then return end
    INV:subItemById(source, itemId, function(ok)
        if ok then
            Core.NotifyTip(source, "You tore the document apart", 3000)
        end
    end, false, 1)
end)
```

```lua
-- client/main.lua (optional — if client event is also used in context)
AddEventHandler("myscript:doc:read", function(args, itemId)
    -- local clientText = args[1]
end)
```

---

## Example 5 — Job-Only Interaction (PolyZone + Permission Check)

Only Sheriff deputies can trigger an evidence locker inside the sheriff's office.

```lua
-- shared/config.lua
Config = {}
Config.Locker = {
    coords = vector3(-277.96, 803.29, 119.44),
    allowedJobs = { "sheriff", "marshal", "deputy" },
}
```

```lua
-- server/main.lua
local Core = exports.vorp_core:GetCore()

RegisterNetEvent("myscript:locker:open", function()
    local source = source
    local user = Core.getUser(source)
    if not user then return end
    local char = user.getUsedCharacter

    local ok = false
    for _, j in ipairs(Config.Locker.allowedJobs) do
        if char.job == j then ok = true; break end
    end

    if not ok then
        Core.NotifyFail(source, "Access denied", "Only law enforcement", 3000)
        return
    end

    -- In a real script, open a custom stash here via vorp_inventory:OpenCustomInventory
    Core.NotifyTip(source, "Locker opened", 3000)
end)
```

```lua
-- client/main.lua
local Lib = Import({"polyzones", "prompts"})
local Zones   = Lib.polyzones.Zones
local Prompts = Lib.prompts.Prompts

local prompt = Prompts:Register({
    text = "Open Locker",
    key = 0xE832C8BB,
    visible = false,
    enabled = false,
    onPressed = function()
        TriggerServerEvent("myscript:locker:open")
    end
})

local locker = Zones.Circle:Create({
    coords = Config.Locker.coords,
    radius = 2.0,
    debug = false,
    onEnter = function()
        prompt:SetVisible(true)
        prompt:SetEnabled(true)
    end,
    onExit = function()
        prompt:SetVisible(false)
        prompt:SetEnabled(false)
    end
})
```

---

## Example 6 — Wrapper Helpers

Small reusable helpers for common patterns. Put these in `shared/utils.lua`.

```lua
-- shared/utils.lua
VorpUtils = {}

-- Promise-ify any callback-based export call
function VorpUtils.promisify(exportFn)
    return function(...)
        local p = promise.new()
        local n = select("#", ...)
        local args = {...}
        args[n + 1] = function(result) p:resolve(result) end
        exportFn(table.unpack(args, 1, n + 1))
        return Citizen.Await(p)
    end
end

-- Server: wrap common inventory exports
if IsDuplicityVersion() then
    local INV = exports.vorp_inventory
    VorpUtils.Inventory = {
        canCarry   = VorpUtils.promisify(function(...) INV:canCarryItem(...) end),
        add        = VorpUtils.promisify(function(...) INV:addItem(...) end),
        sub        = VorpUtils.promisify(function(...) INV:subItem(...) end),
        count      = VorpUtils.promisify(function(...) INV:getItemCount(...) end),
        giveWeapon = VorpUtils.promisify(function(...) INV:giveWeapon(...) end),
    }

    -- Require a character (errors to falsey)
    function VorpUtils.withCharacter(source, fn)
        local Core = exports.vorp_core:GetCore()
        local user = Core.getUser(source)
        if not user then return nil end
        local char = user.getUsedCharacter
        if not char then return nil end
        return fn(char)
    end

    function VorpUtils.moneyCheck(source, amount)
        return VorpUtils.withCharacter(source, function(c)
            return c.money >= amount, c
        end)
    end
end
```

Usage from a server event:

```lua
RegisterNetEvent("example:buyGun", function(weapon)
    local source = source
    local canAfford, char = VorpUtils.moneyCheck(source, 500)
    if not canAfford then return end

    if VorpUtils.Inventory.canCarry(source, weapon, 1) then
        char.deductMoney(500)
        -- createWeapon here or your logic
    end
end)
```

---

## Example 7 — Skill XP Gain + Level-Up Reaction

When a player skins an animal, add Hunting XP and notify on level-up.

```lua
-- server/main.lua
local Core = exports.vorp_core:GetCore()

local XpPerAnimal = { deer = 25, cow = 40, boar = 30 }

RegisterNetEvent("myscript:skinnedAnimal", function(animalType)
    local source = source
    local xp = XpPerAnimal[animalType] or 10

    local user = Core.getUser(source)
    if not user then return end
    local char = user.getUsedCharacter

    local beforeLevel = 0
    local skills = char.skills
    if skills and skills.Hunting then
        beforeLevel = skills.Hunting.Level
    end

    char.addXp(xp)  -- XP goes into Hunting depending on Core config or
                    -- call a skill-specific add if Core exposes it

    -- After addXp, skills are refreshed on the char (re-read):
    local newSkills = char.skills
    if newSkills and newSkills.Hunting and newSkills.Hunting.Level > beforeLevel then
        Core.NotifySimpleTop(source,
            "Hunting Level Up!",
            ("You are now level %d"):format(newSkills.Hunting.Level),
            5000
        )
        -- Level-up reward
        char.addMoney(20)
    end

    Core.NotifyTip(source, ("+%d Hunting XP"):format(xp), 2000)
end)
```

---

## Example 8 — Input Dialog (vorp_inputs:advancedInput)

Prompt player for amount before giving cash to an NPC.

```lua
-- client/main.lua
RegisterNetEvent("myscript:openDonatePrompt", function()
    local input = {
        type = "enableinput",
        inputType = "input",
        button = "DONATE",
        placeholder = "Amount ($)",
        style = "block",
        attributes = {
            inputHeader = "Donate to Camp",
            type = "number",
            pattern = "[0-9]",
            title = "Numbers only",
        }
    }
    local result = exports.vorp_inputs:advancedInput(input)
    local amount = tonumber(result)
    if amount and amount > 0 then
        TriggerServerEvent("myscript:donate", amount)
    end
end)
```

```lua
-- server/main.lua
RegisterNetEvent("myscript:donate", function(amount)
    local source = source
    local Core = exports.vorp_core:GetCore()
    local user = Core.getUser(source)
    if not user then return end
    local char = user.getUsedCharacter

    if char.money >= amount then
        char.deductMoney(amount)
        Core.NotifyTip(source, ("Donated $%d to the camp"):format(amount), 4000)
        -- Persist camp fund to DB, etc.
    else
        Core.NotifyFail(source, "Cannot donate", "Not enough money", 3000)
    end
end)
```

---

## Summary Table

| Task | API surface |
|---|---|
| Character data | `Core.getUser(source).getUsedCharacter` |
| Economy | `char.addMoney / deductMoney / addGold` |
| Add/remove items | `exports.vorp_inventory:{add,sub}Item` |
| Weight check | `canCarryItem` before every `addItem` |
| Reusable UI for E interactions | VORP Lib `prompts` module |
| Proximity zones | VORP Lib `polyzones` or `points` modules |
| Custom item right-click | Metadata `context` array + whitelist exports |
| Number / text dialogs | `exports.vorp_inputs:advancedInput` |
| Notifications | `Core.Notify*` (18+ variants) |
