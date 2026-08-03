---
title: VORP Overview
category: Frameworks
tags:
  - vorp
  - framework
  - redm
framework:
  - vorp
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/
  - https://docs.vorp-core.com/introduction
  - https://vorp-core.com/
---

# VORP Overview

VORP Core (**V**ersatile **O**pen **R**oleplay **P**latform) is the leading open-source roleplay framework built specifically for RedM. It provides the foundation for Wild West RP servers with multi-character support, dual-currency economy, weight-based inventory, and a fully-documented modular API.

VORP is the RedM ecosystem equivalent of ESX or QBCore in FiveM — the most widely-adopted framework with the largest script ecosystem.

## Core Features

| Feature | Description |
|---|---|
| Multi-character | Up to 5 characters per account with full appearance customization |
| Dual currency | Dollars ($) and Gold coins, period-appropriate for 1898 |
| Inventory system | Weight-based with weapons, ammo, money, and metadata support |
| Skill system | XP-based skills (Crafting, Hunting, Fishing, etc.) with levels |
| Character creation | In-depth face, hair, body, clothing, and overlay options |
| Outfits system | Save/load character outfits with a server-side menu |
| Job & grade system | Multiple jobs per character with configurable permission groups |
| Database storage | MySQL / oxmysql persistent storage for all player data |
| Developer API | Shared `GetCore()` export, callbacks, webhooks, and module library |
| VORP Lib | Instance-based component library (blips, prompts, zones, entities) with auto-cleanup |

## Architecture

### Core Resources

A standard VORP installation consists of these base resources loaded in order:

```
ensure oxmysql           -- MySQL async database driver
ensure vorp_core         -- Core framework (users, chars, jobs, economy)
ensure vorp_character    -- Character creation, selection, outfits
ensure vorp_inventory    -- Inventory, items, weapons, ammo, stashes
ensure vorp_progressbar  -- Progress bar UI (used by crafting, etc.)
ensure vorp_inputs       -- Advanced input dialogs (text, number, textarea)
ensure vorp_lib          -- Modular component library (optional but recommended)
```

### Layer Model

```
┌───────────────────────────────────────────────┐
│              Client Scripts (your scripts)    │
│  ┌─────────┐  ┌──────────┐  ┌─────────────┐  │
│  │ prompts │  │  blips   │  │ polyzones   │  │  ← vorp_lib modules
│  └─────────┘  └──────────┘  └─────────────┘  │
├───────────────────────────────────────────────┤
│          Server Scripts (your scripts)        │
│  ┌──────────────┐  ┌─────────────────────┐    │
│  │  GetCore()   │  │  Inventory exports  │    │
│  └──────────────┘  └─────────────────────┘    │
├───────────────────────────────────────────────┤
│               VORP Resources                  │
│  vorp_core  |  vorp_character  |  vorp_inv    │
├───────────────────────────────────────────────┤
│              oxmysql + Database               │
└───────────────────────────────────────────────┘
```

### Data Model (Simplified)

```
User (account)
 ├── identifier       (license:xxx or discord:xxx)
 ├── group            (user, admin, etc.)
 ├── whitelist        (boolean)
 └── Characters[]     (up to Config.MaxCharacters)
      ├── charIdentifier   (unique per character)
      ├── firstname, lastname
      ├── job, jobGrade, jobLabel
      ├── multiJobs[]      ({job, grade, label})
      ├── money, gold, rol, xp
      ├── skills[]         ({Exp, Level})
      ├── skin, comps, compTints
      └── invCapacity
```

## Ecosystem

VORP has an extensive ecosystem of compatible scripts from the community:

### Official / Core Resources

- **vorp_core** — Users, characters, jobs, economy, callbacks, webhooks
- **vorp_character** — Character creator, selector, outfits, barber
- **vorp_inventory** — Items, weapons, ammo, custom inventories, crafting
- **vorp_inputs** — Advanced text/number input dialogs
- **vorp_progressbar** — Progress bar UI component
- **vorp_lib** — Instance-based component library (entities, blips, prompts, zones, points, streaming, logger)

### Common Community Resources

- **vorp_admin** — Admin menu with teleport, revive, spectate, noclip
- **vorp_housing** — Housing / property system
- **vorp_shops** — General stores, weapon shops, tailor
- **vorp_banking** — Bank and ATM system
- **vorp_fishing** — Fishing minigame
- **vorp_hunting** — Hunting / skinning system
- **vorp_heists** — Bank / train heists
- **vorp_voice** — Proximity voice chat (SaltyChat / PMA-Voice integration)

## Quick API Cheat Sheet

```lua
-- SHARED: Get the core table
local Core = exports.vorp_core:GetCore()

-- SERVER: Get user + character
local user = Core.getUser(source)
if not user then return end
local char = user.getUsedCharacter
print(char.firstname, char.lastname, "$"..char.money, "Gold:"..char.gold)

-- SERVER: Economy
char.addMoney(100)
char.deductMoney(25)
char.addGold(5)

-- SERVER: Inventory (async via callback or promise)
exports.vorp_inventory:addItem(source, "bread", 3, {}, function(success)
    -- done
end)

-- CLIENT: Character selected event
RegisterNetEvent("vorp:SelectedCharacter", function(charid)
    -- init client-side logic
end)

-- VORP LIB: Import a module (add shared_script "@vorp_lib/import.lua" first)
local Prompts = Import("prompts").Prompts
```

## This Documentation

| Page | Content |
|---|---|
| [setup.md](file:///d:/Playground/redm-knowledge-base/docs/frameworks/vorp/setup.md) | Installation, fxmanifest, config, server.cfg startup order |
| [api.md](file:///d:/Playground/redm-knowledge-base/docs/frameworks/vorp/api.md) | Full Core, Lib, Character, and Inputs API reference |
| [inventory.md](file:///d:/Playground/redm-knowledge-base/docs/frameworks/vorp/inventory.md) | Inventory exports (items, weapons, ammo, metadata, events) |
| [examples.md](file:///d:/Playground/redm-knowledge-base/docs/frameworks/vorp/examples.md) | Complete working code snippets and patterns |

## Community & Links

- **Official docs:** https://docs.vorp-core.com/
- **GitHub org:** https://github.com/VORPCORE
- **Discord:** https://discord.gg/vorp
