---
title: VORP Setup
category: Frameworks
tags:
  - vorp
  - setup
  - framework
framework:
  - vorp
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/introduction
  - https://docs.vorp-core.com/api-reference/lib
---

# VORP Setup

## Prerequisites

Before installing VORP, confirm the following are installed and working:

| Requirement | Purpose |
|---|---|
| RedM Server | The game server runtime (requires RDR2 installed locally for artifacts) |
| MySQL / MariaDB | Persistent storage for all players, characters, items, economy |
| **oxmysql** | Async MySQL driver resource (required; replacement for `mysql-async`) |
| Lua knowledge | Basic familiarity with Lua 5.4 and FiveM/RedM scripting patterns |

### server.cfg: Database & Base

```cfg
set mysql_connection_string "mysql://redm_user:your_password@localhost/redm_db?charset=utf8mb4"

# Steam / Discord (optional but recommended for identifiers)
set steam_webApiKey "YOUR_STEAM_KEY"

# Ensure database driver loads FIRST
ensure oxmysql
```

---

## Installation

### 1. Install VORP Core

Clone or download the core resources into your server `resources/[core]` folder:

```bash
cd resources/[core]
git clone https://github.com/VORPCORE/vorp-core-lua.git vorp_core
git clone https://github.com/VORPCORE/vorp_character-lua.git vorp_character
git clone https://github.com/VORPCORE/vorp_inventory-lua.git vorp_inventory
git clone https://github.com/VORPCORE/vorp_progressbar-lua.git vorp_progressbar
git clone https://github.com/VORPCORE/vorp_inputs-lua.git vorp_inputs
git clone https://github.com/VORPCORE/vorp_lib.git vorp_lib
```

### 2. Import SQL Databases

Each core resource ships a `database/` folder with a `.sql` schema. Import them **in order** because of foreign key relationships:

```bash
mysql -u redm_user -p redm_db < resources/[core]/vorp_core/database/vorp_core.sql
mysql -u redm_user -p redm_db < resources/[core]/vorp_character/database/vorp_character.sql
mysql -u redm_user -p redm_db < resources/[core]/vorp_inventory/database/vorp_inventory.sql
```

### 3. server.cfg Resource Order

Order is critical. Resources that depend on VORP Core **must** start after it.

```cfg
# ====== DATABASE ======
ensure oxmysql

# ====== VORP CORE (order matters) ======
ensure vorp_core          # 1. Core framework
ensure vorp_character     # 2. Character creation / selection (depends on core)
ensure vorp_inventory     # 3. Inventory (depends on core + character)
ensure vorp_progressbar   # 4. Progress bar UI (optional dependency for inventory crafting/repair)
ensure vorp_inputs        # 5. Input dialogs (used by many scripts)
ensure vorp_lib           # 6. Modular library (optional but recommended)

# ====== VORP DEPENDENT SCRIPTS ======
# (your custom scripts, vorp_admin, vorp_shops, vorp_housing, etc. go here)
ensure vorp_admin
ensure my_custom_script
```

### 4. Populate Items Table

VORP Inventory requires items to be registered in the `items` database table before they can be used. Example seed:

```sql
INSERT INTO items (item, label, weight, description, type) VALUES
('bread',     'Bread',           200, 'A loaf of fresh bread',          'item_standard'),
('water',     'Water Bottle',    500, 'Clean drinking water',           'item_standard'),
('bandage',   'Bandage',         100, 'A basic medical bandage',        'item_standard'),
('whiskey',   'Whiskey',         400, 'A bottle of fine whiskey',       'item_standard'),
('tobacco',   'Tobacco',          50, 'Dried tobacco leaves',           'item_standard'),
('rope',      'Rope',            300, 'A sturdy length of rope',        'item_standard'),
('meat_raw',  'Raw Meat',        500, 'Uncooked animal meat',           'item_standard'),
('meat_cooked','Cooked Meat',    400, 'Cooked, safe-to-eat meat',       'item_standard'),
('revolver',  'Cattleman Revolver', 1200, 'A reliable sidearm',       'item_weapon'),
('ammo_revolver','Revolver Ammo',  20, 'Six rounds of .44 ammo',       'item_ammo');
```

---

## Configuration

### vorp_core/config.lua

Core behaviour settings:

```lua
Config = {}

-- Characters
Config.MaxCharacters   = 5            -- Max per account
Config.StartingMoney   = 100.00       -- Dollars
Config.StartingGold    = 5.00         -- Gold coins
Config.StartingROL     = 0            -- RP points

-- Spawn
Config.DefaultSpawn = {
    x = -279.22, y = 788.42, z = 118.35, heading = 180.0  -- Valentine
}

-- Save / death
Config.SaveInterval   = 5             -- minutes between auto-saves
Config.RespawnTime    = 60            -- seconds before respawn button enables
Config.RespawnCost    = 5.00          -- dollars deducted on death

-- Whitelist
Config.WhitelistEnabled = false

-- XP / Skill defaults (see config/skills.lua for full list)
Config.Skills = {
    Crafting = { Label = "Crafting", MaxLevel = 20, XPToNext = 100 },
    Hunting  = { Label = "Hunting",  MaxLevel = 20, XPToNext = 100 },
    Fishing  = { Label = "Fishing",  MaxLevel = 20, XPToNext = 100 },
}
```

### vorp_character/config.lua

Character creation limits and appearance options:

```lua
Config = {}
Config.MinFirstNameLength = 2
Config.MaxFirstNameLength = 20
Config.MinLastNameLength  = 2
Config.MaxLastNameLength  = 20
Config.MinAge = 18
Config.MaxAge = 80

Config.Genders = {
    { label = "Male",   value = "male",   model = "mp_male"   },
    { label = "Female", value = "female", model = "mp_female" },
}

Config.AppearanceOptions = {
    face      = true,
    hair      = true,
    beard     = true,
    body      = true,
    overlays  = true,
    clothing  = true,
}
```

### vorp_inventory/config.lua

```lua
Config = {}

Config.InventoryType  = "weight"       -- "weight" or "slot"
Config.MaxWeight      = 24000          -- grams (24 kg)
Config.HotbarSlots    = 5              -- quick-use slots
Config.MaxStack       = 50             -- default item stack limit
Config.DropOnDeath    = false          -- drop items on death
Config.UsePickupAnimation = true
Config.OpenKey        = "TAB"

-- Starting items (new characters)
Config.StartingItems = {
    { item = "bread",     count = 5 },
    { item = "water",     count = 3 },
    { item = "bandage",   count = 2 },
}
```

---

## Developing a Script That Uses VORP

### fxmanifest.lua

Every resource that uses VORP should declare dependencies and optionally import VORP Lib:

```lua
fx_version 'adamant'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM, and I am aware my resources *will* become incompatible once RedM ships.'

author 'Your Name'
description 'My custom VORP script'
version '1.0.0'

-- CRITICAL: ensures VORP loads before your script
dependencies {
    'vorp_core',
    'vorp_character',
    'vorp_inventory',
}

-- Optional: import vorp_lib modules (enables Import() function)
shared_script '@vorp_lib/import.lua'

-- Your actual files
shared_scripts {
    'shared/config.lua',
}

client_scripts {
    'client/*.lua',
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',  -- if you need DB queries
    'server/*.lua',
}

ui_page 'html/index.html'
files {
    'html/index.html',
    'html/style.css',
    'html/script.js',
}
```

### Core Access Pattern (Shared / Both Sides)

```lua
-- Boilplate at the top of any server/client file
local Core = exports.vorp_core:GetCore()
```

### Character-Ready Guard

Always gate logic that needs a logged-in character:

```lua
-- Server-side gate
local function requireCharacter(source, fn)
    local user = Core.getUser(source)
    if not user then return false end
    local char = user.getUsedCharacter
    if not char then return false end
    return fn(char)
end

-- Example use with an event
RegisterNetEvent("myscript:doSomething", function()
    local source = source
    requireCharacter(source, function(char)
        -- char is guaranteed valid here
        char.addMoney(10)
        Core.NotifyTip(source, "Earned $10", 3000)
    end)
end)
```

```lua
-- Client-side gate using the SelectedCharacter event
local characterReady = false
RegisterNetEvent("vorp:SelectedCharacter", function(charid)
    characterReady = true
    -- Initialize client-only things (blips, prompts, zones, ...)
end)

-- Later, before doing client work:
if not characterReady then return end
```

---

## Common Setup Pitfalls

| Symptom | Cause | Fix |
|---|---|---|
| `GetCore is nil` when calling exports.vorp_core | `vorp_core` not ensured before your script, or `dependencies` missing | Add `dependencies { 'vorp_core' }` in fxmanifest |
| Character always returns `nil` on event | Code runs before player selected a character | Use `vorp:SelectedCharacter` event gate |
| Item gives "not found in DB" | Item not inserted into `items` table | INSERT into `items` table with correct `type` |
| Inventory won't open | `vorp_inventory` started before `vorp_character` | Correct server.cfg order: core → character → inventory |
| oxmysql connection error | Wrong `mysql_connection_string` or DB user permissions | Verify string format, charset=utf8mb4, user has ALL PRIVILEGES |
| `Import` global is undefined | `@vorp_lib/import.lua` not in shared_scripts | Add `shared_script '@vorp_lib/import.lua'` to fxmanifest |

---

## Updating VORP

1. Stop the server.
2. Git pull each repo (or download fresh files):
   ```bash
   cd resources/[core]/vorp_core
   git pull origin main
   cd ../vorp_inventory && git pull origin main
   ```
3. Check for SQL migration files in each resource's `database/` folder. Apply any new `.sql` files.
4. Compare `config.lua` against `config.example.lua` (if provided) for new keys.
5. Start the server.
