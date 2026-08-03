---
title: VORP Inventory
category: Frameworks
tags:
  - vorp
  - inventory
  - items
framework:
  - vorp
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/inventory
  - https://docs.vorp-core.com/api-reference/lib
---

# VORP Inventory

VORP Inventory (`vorp_inventory`) is the official item, weapon, ammo, and custom-inventory resource for VORP Core. It exposes a full server-side export surface and a smaller set of client-side exports. Most server exports support both callback-style and promise-await (async) usage.

Export target: `exports.vorp_inventory:<exportName>(...)`

---

## Client Exports

Use these from any **client** resource.

### Inventory UI Control

```lua
-- Open the main inventory
exports.vorp_inventory:openInventory()

-- Close the inventory
exports.vorp_inventory:closeInventory()

-- Toggle (open if closed, close if open)
exports.vorp_inventory:toggleInventory()

-- Check if currently open
local isOpen = exports.vorp_inventory:isInventoryOpen()

-- Prevent inventory from opening on client (e.g. during a cutscene)
exports.vorp_inventory:setInventoryDisabled(true)   -- disable
exports.vorp_inventory:setInventoryDisabled(false)  -- re-enable
```

### Client Inventory Lookups

```lua
-- All inventory item definitions (name -> {label, weight, desc, ...})
local items = exports.vorp_inventory:getInventoryItems()

-- Single inventory item definition by name
local water = exports.vorp_inventory:getInventoryItem("water")
if water then
    print(water.label, water.weight)
end

-- Resolve server-side item metadata by name(s)
local item = exports.vorp_inventory:getServerItem("bread")
print(item.label, item.desc)

-- Batch resolve
local batch = exports.vorp_inventory:getServerItem({ "bread", "water", "bandage" })
for _, v in ipairs(batch) do print(v.label) end
```

---

## Server Exports — Items

Use from any **server** resource. Most methods are asynchronous; pass a callback or wrap in `Citizen.Await` + `promise`.

### Capacity / Carry Checks

#### canCarryItem

```lua
-- Callback style
exports.vorp_inventory:canCarryItem(source, "bread", 5, function(can)
    if can then
        -- proceed with addItem
    else
        -- too heavy / inventory full
    end
end)

-- Async (promise) style
local function canCarry(source, item, amount)
    local p = promise.new()
    exports.vorp_inventory:canCarryItem(source, item, amount, function(r) p:resolve(r) end)
    return Citizen.Await(p)
end

if canCarry(source, "bread", 5) then
    -- ok
end
```

### Item Getters

#### getUserInventoryItems

```lua
exports.vorp_inventory:getUserInventoryItems(source, function(items)
    for _, item in ipairs(items) do
        -- item = { id, name, label, count, metadata, type, weight, desc, ... }
    end
end)
```

#### getItemCount

Get the total stack count of an item. `percentage` filters by durability/degradation: `nil` = any, `0` = expired only, `>0` = percentage >= value.

```lua
exports.vorp_inventory:getItemCount(source, function(count)
    print("water x"..count)
end, "water")

-- With metadata filter
exports.vorp_inventory:getItemCount(source, function(count) end,
    "custom_item", { quality = "perfect" })
```

#### getItem

Full item row by name (and optional metadata / percentage filter).

```lua
exports.vorp_inventory:getItem(source, "bread", function(item)
    if item then
        print(item.id, item.label, "x"..item.count, item.weight)
    end
end)
```

#### getItemById

Fetch a specific inventory slot entry by its database `id`.

```lua
exports.vorp_inventory:getItemById(source, itemId, function(row)
    -- row = { id, label, name, metadata, group, type, count, limit,
    --         canUse, weight, desc, percentage, description }
end)
```

#### getItemDB

Look up the *database* item definition (not player-owned) — useful for labels, weights, default metadata.

```lua
exports.vorp_inventory:getItemDB("bread", function(def)
    print(def.label, def.weight, def.type)
end)
```

### Item Setters

#### addItem

Add items to a player.

```lua
exports.vorp_inventory:addItem(source, "bread", 3, {}, function(success)
    if success then
        Core.NotifyTip(source, "Given 3 bread", 2000)
    end
end)

-- With custom metadata + label override
local metadata = {
    label = "Signed Contract",
    description = "Property transfer between John and Arthur",
    image = "custom_contract.png",
}
exports.vorp_inventory:addItem(source, "document", 1, metadata, function(ok) end)

-- Last param (event = true) DISABLES the OnItemCreated event from firing
exports.vorp_inventory:addItem(source, "bread", 1, {}, cb, true)
```

#### subItem

Remove items from a player by name.

```lua
exports.vorp_inventory:subItem(source, "bread", 2, {}, function(success)
    if success then
        -- removed
    end
end)
```

Event suppression: 6th param = `true` prevents `OnItemRemoved` from firing.
Percentage filter: 7th param controls degradation target for removal.

```lua
exports.vorp_inventory:subItem(source, item, amt, meta, cb, fireEvent, percentage)
```

#### subItemById

Remove by the actual slot `id` (preferred for precise removal).

```lua
-- (v3.9+)
exports.vorp_inventory:subItemById(source, itemId, function(ok) end)

-- With fire-event flag + partial amount
exports.vorp_inventory:subItemById(source, itemId, cb, false, 2)
```

---

## Item Metadata & Context Menus

VORP Inventory supports per-stack metadata that overrides labels, images, and adds right-click context actions to items.

### Reserved Metadata Keys

```lua
local metadata = {
    label       = "Override label",            -- replace DB label in UI
    image       = "custom.png",                -- image file in vorp_inventory/html/img/items/
    description = "Longer description text",   -- replaces DB desc
    weight      = 500,                         -- custom stack weight
    tolltip     = "Extra tooltip text",        -- extra hover info
    useExpired  = true,                        -- allow using degraded/expired items
    context     = {
        {
            text  = "READ CONTRACT",
            close = true,                       -- close inventory on click
            event = {
                client = "myscript:readDoc",    -- client event to fire
                server = "myscript:readDoc",    -- server event to fire (optional)
            },
            arguments = { "extra_arg" }         -- optional args
        },
        {
            text  = "TEAR UP",
            event = { server = "myscript:destroyDoc" },
        }
    }
}
```

### setItemMetadata

Split/merge/modify stacks with new metadata:

```lua
-- Case 1: amount < stack size => split off into new stack with new meta
exports.vorp_inventory:setItemMetadata(source, itemId, newMeta, 2, function(ok) end)

-- Case 2: amount >= stack size, same meta => merge
exports.vorp_inventory:setItemMetadata(source, itemId, newMeta, 99, function(ok) end)
```

### Context Menu Event Whitelisting (SECURITY)

All server-side events called via `context` metadata **must be whitelisted**, otherwise they are ignored.

```lua
-- Whitelist one event
exports.vorp_inventory:addAllowedContextMenuEvent(
    "myscript:readDoc", GetCurrentResourceName()
)

-- Whitelist multiple
exports.vorp_inventory:addAllowedContextMenuEvent({
    "myscript:readDoc",
    "myscript:destroyDoc",
}, GetCurrentResourceName())

-- Remove whitelist
exports.vorp_inventory:removeAllowedContextMenuEvent(
    "myscript:readDoc", GetCurrentResourceName()
)
exports.vorp_inventory:removeAllowedContextMenuEvents({
    "myscript:readDoc",
}, GetCurrentResourceName())
```

Client handler receives `(arguments, itemId)`:

```lua
-- Client
AddEventHandler("myscript:readDoc", function(args, itemId)
    print("Read document, item id:", itemId)
end)

-- Server
AddEventHandler("myscript:readDoc", function(source, args, itemId)
    -- trusted because whitelisted
end)
```

---

## Server Exports — Weapons

### Weapon Getters

#### getUserInventoryWeapons

```lua
exports.vorp_inventory:getUserInventoryWeapons(source, function(weapons)
    for _, w in ipairs(weapons) do
        -- w = { id, name, propietary, used, desc, group, source,
        --       label, serial_number, custom_label, custom_desc }
    end
end)
```

#### getUserWeapon

Single weapon by `weaponId`:

```lua
exports.vorp_inventory:getUserWeapon(source, function(w)
    print(w.name, w.label, "serial:"..w.serial_number)
end, weaponId)
```

#### canCarryWeapons

```lua
exports.vorp_inventory:canCarryWeapons(source, 1, function(can) end, "WEAPON_REVOLVER_CATTLEMAN")
```

### Weapon Setters

#### createWeapon

Spawn a new weapon into inventory:

```lua
exports.vorp_inventory:createWeapon(
    source,
    "WEAPON_REVOLVER_CATTLEMAN",         -- weaponName
    "AMMO_REVOLVER",                     -- ammo type string
    {},                                  -- bullet component table
    {},                                  -- comps
    function(ok) end,                    -- callback
    nil,                                 -- wepid (internal, leave nil)
    "CR-1337-AU",                        -- custom serial (optional)
    "Arthur's Peacemaker",               -- custom label (optional)
    "Etched with a stag."                -- custom desc  (optional)
)
```

#### subWeapon / deleteWeapon

```lua
-- Remove from inventory (returns to DB pool? — no, just removes from player)
exports.vorp_inventory:subWeapon(source, weaponId, function(ok) end)

-- Permanently delete from DB
exports.vorp_inventory:deleteWeapon(source, weaponId, function(ok) end)
```

#### giveWeapon

Transfer to another player:

```lua
exports.vorp_inventory:giveWeapon(source, weaponId, targetSource, function(ok) end)
```

#### Weapon customisation

```lua
exports.vorp_inventory:setWeaponCustomLabel(source, weaponId, "New Label", cb)
exports.vorp_inventory:setWeaponCustomDesc(source, weaponId, "New Desc", cb)
exports.vorp_inventory:setWeaponSerialNumber(source, weaponId, "SN-0001", cb)
```

### Weapon Components

```lua
-- Get
exports.vorp_inventory:getWeaponComponents(source, weaponId, function(comps) end)

-- Add single
exports.vorp_inventory:addWeaponComponent(
    source, weaponId, "COMPONENT_MELEE_KNIFE_SCOOP", "slot_category", cb
)

-- Add multiple: key = component hash/name, value = category
exports.vorp_inventory:addWeaponComponents(source, weaponId, {
    ["COMPONENT_1"] = "barrel",
    ["COMPONENT_2"] = "grip",
}, cb)

-- Remove single / multiple
exports.vorp_inventory:subWeaponComponent(source, weaponId, component, category, cb)
exports.vorp_inventory:subWeaponComponents(source, weaponId, compsTable, cb)
```

---

## Server Exports — Ammo

```lua
-- Get all ammo entries for player
exports.vorp_inventory:getUserAmmo(source, function(ammo)
    for type, qty in pairs(ammo) do
        print(type, qty)
    end
end)

-- Add bullets
exports.vorp_inventory:addBullets(source, "AMMO_REVOLVER", 12, function(ok) end)

-- Subtract bullets from a specific weapon
exports.vorp_inventory:subBullets(weaponId, "AMMO_REVOLVER", 6, function(ok) end)

-- Read bullets in a weapon
exports.vorp_inventory:getWeaponBullets(source, weaponId, function(amt) end)

-- Nuke all ammo (e.g. on arrest)
exports.vorp_inventory:removeAllUserAmmo(source)
```

---

## Inventory Events

Subscribe to these events in your scripts for reaction logic.

### Item Events (Server)

```lua
-- Fired when an item is *created* (picked up, given, added)
AddEventHandler("vorp_inventory:OnItemCreated", function(source, item, name, amount)
    -- item = full row
    -- name = item key
    -- amount = count added
end)

-- Fired when an item is *removed* (dropped, used, given, subtracted)
AddEventHandler("vorp_inventory:OnItemRemoved", function(source, name, amount, metadata, itemId)
    -- name, amount, metadata, slot itemId
end)

-- Item used (from inventory UI)
AddEventHandler("vorp_inventory:useItem", function(source, item)
    -- e.g. consumable logic
end)
```

### Inventory State Events (Client + Server)

```lua
-- Client: inventory was just opened
RegisterNetEvent("vorp_inventory:opened", function() end)

-- Client: inventory was just closed
RegisterNetEvent("vorp_inventory:closed", function() end)

-- Server/Client: custom inventory slot changes
AddEventHandler("vorp_inventory:customInventoryChanged", function(invId, source, action) end)
AddEventHandler("vorp_inventory:itemAddedToCustom",   function(invId, source, itemName, amount) end)
AddEventHandler("vorp_inventory:itemRemovedFromCustom", function(invId, source, itemName, amount) end)
```

### Block Inventory Open

Prevent a player from opening the inventory at all:

```lua
-- Server
TriggerClientEvent("vorp_inventory:serverLockInventory", source, true)
TriggerClientEvent("vorp_inventory:serverLockInventory", source, false)

-- Client
TriggerEvent("vorp_inventory:clientLockInventory", true)
TriggerEvent("vorp_inventory:clientLockInventory", false)
```

---

## StateBags

```lua
-- Weapon the player currently has equipped (client-side, replicated)
local equipped = LocalPlayer.state.GetEquippedWeaponData
-- { id, name, label, ammo, components, ... } or nil

-- Custom inventory currently "in use" by player
local inv = LocalPlayer.state.CustomInvInUse
```

---

## Best Practices

1. **Always `canCarryItem` before `addItem`** — never assume the inventory fits items.
2. **Prefer callbacks over raw events** — the deprecated `vorpCore:*` events still work but exports are type-safer.
3. **Whitelist all context-menu server events** — unwhitelisted server events silently fail.
4. **Never trust the client** for counts/metadata — always re-check on server with `getItemCount` before rewarding.
5. **Handle `percentage`** — degradable items with `percentage = 0` are expired; filter when giving rewards.
