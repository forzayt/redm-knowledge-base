---
title: VORP API
category: Frameworks
tags:
  - vorp
  - api
  - framework
framework:
  - vorp
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.vorp-core.com/api-reference/core
  - https://docs.vorp-core.com/api-reference/lib
  - https://docs.vorp-core.com/api-reference/characters
  - https://docs.vorp-core.com/api-reference/inputs
---

# VORP API

## Core Export

### GetCore

Shared export that returns the core table containing all getters and setters for VORP Core. Some methods are client-side only, some are server-side only.

```lua
local Core = exports.vorp_core:GetCore()
```

Return: `table` - The core table.

---

## Register Server Jobs

Allows registration of valid jobs and grades to prevent invalid job assignments. Jobs are stored in `config/jobs.lua` in vorp_core.

### RegisterJobs (Server)

```lua
local jobsData <const> = {
  jobname = {
    groups = {"admin"},
    privateJob = true,
    grades = {
      [1] = {
        label = "Grade Label",
        privateGrade = true
      }
    }
  }
}

if Core.RegisterJobs then
  Core.RegisterJobs(jobsData, GetCurrentResourceName())
else
  print("Core.RegisterJobs is not available - update vorp_core")
end
```

| Parameter | Type | Required | Description |
|---|---|---|---|
| jobsData | table | yes | Job definitions keyed by job name |
| resourceName | string | yes | Resource name for debug output |

**jobsData keys:**
- `groups` (array, optional): Only users in these groups can assign this job
- `privateJob` (boolean): If true, no one can assign this job
- `grades` (table): Grade number as key, containing `label` and `privateGrade`

---

## Notifications

Notification methods support `-1` as duration for permanent display. Clear with `UiFeedClearChannel`.

### Client-Side Notifications

```lua
-- Left notification with icon
Core.NotifyLeft("title", "subtitle", "dict", "icon", 4000, "color")

-- Tip-style notifications
Core.NotifyTip("title", 4000)
Core.NotifyRightTip("title", 4000)
Core.NotifyObjective("title", 4000)

-- Top notifications
Core.NotifyTop("title", "location", 4000)
Core.NotifySimpleTop("title", "subtitle", 4000)
Core.NotifyOneSimpleTop("title", 4000)
Core.NotifyThreeSimpleTop("title", "description", "second_description", 4000)

-- Advanced with quality stars (animal pelts, etc.)
Core.NotifyAvanced("title", "dict", "icon", "color", 4000, quality, showQuality)

-- Other positions
Core.NotifyCenter("title", 4000)
Core.NotifyBottomRight("title", 4000)

-- Status notifications
Core.NotifyFail("title", "subtitle", 4000)
Core.NotifyDead("title", "audioref", "audioname", 4000)
Core.NotifyUpdate("title", "subtitle", 4000)
Core.NotifyWarning("title", "subtitle", "audioref", "audioname", 4000)
Core.NotifyLeftRank("title", "subtitle", "dict", "icon", 4000, "color")

-- Interactive (supports -1 duration for infinite)
Core.NotifyLeftInteractive("title", "subtitle", "dict", "icon", 4000, "color")
```

### Server-Side Notifications

All server notification methods take `source` (player ID) as the first parameter:

```lua
Core.NotifyTip(source, "title", 4000)
Core.NotifyLeft(source, "title", "subtitle", "dict", "icon", 4000, "color")
Core.NotifyRightTip(source, "title", 4000)
Core.NotifyObjective(source, "title", 4000)
Core.NotifyTop(source, "title", "location", 4000)
Core.NotifySimpleTop(source, "title", "subtitle", 4000)
Core.NotifyAvanced(source, "title", "dict", "icon", "color", showQuality, quality, 4000)
Core.NotifyCenter(source, "title", 4000)
Core.NotifyBottomRight(source, "title", 4000)
Core.NotifyFail(source, "title", "subtitle", 4000)
Core.NotifyDead(source, "title", "audioref", "audioname", 4000)
Core.NotifyUpdate(source, "title", "subtitle", 4000)
Core.NotifyWarning(source, "title", "subtitle", "audioref", "audioname", 4000)
Core.NotifyLeftRank(source, "title", "subtitle", "dict", "icon", 4000, "color")
Core.NotifyOneSimpleTop(source, "title", 4000)
Core.NotifyThreeSimpleTop(source, "title", "description", "second_description", 4000)
Core.NotifyLeftInteractive(source, "title", "subtitle", "dict", "icon", 4000, "color")
```

---

## Characters

Access character data via the User object.

### getUsedCharacter (Server)

```lua
local user = Core.getUser(source)
if not user then return end

local character = user.getUsedCharacter
```

### Character Getters

All properties are read directly from the character object:

```lua
local identifier     = character.identifier      -- string: unique account identifier
local charIdentifier = character.charIdentifier   -- string: unique character ID
local group          = character.group            -- string: permission group
local multiJobs      = character.multiJobs        -- table: {job, grade, label} per job
local job            = character.job              -- string: current job name
local jobGrade       = character.jobGrade         -- integer: current job grade
local jobLabel       = character.jobLabel         -- string: job display label
local money          = character.money            -- number: cash on hand
local gold           = character.gold             -- number: gold on hand
local rol            = character.rol              -- number: rol (roleplay points)
local xp             = character.xp               -- number: experience points
local firstname      = character.firstname        -- string
local lastname       = character.lastname         -- string
local status         = character.status           -- table: hunger, thirst, etc.
local coords         = character.coords           -- table: last saved position
local isdead         = character.isdead           -- boolean: death state
local skin           = character.skin             -- table: appearance data
local comps          = character.comps            -- table: clothing components
local compTints      = character.compTints        -- table: component tints
local age            = character.age              -- number
local gender         = character.gender           -- string: "male" / "female"
local charDescription = character.charDescription -- string: bio
local nickname       = character.nickname         -- string
local invCapacity    = character.invCapacity      -- number: max inventory weight
local skills         = character.skills           -- table: {Crafting = {Exp, Level}, ...}
```

**Skills example:**

```lua
local skills = character.skills
local skill = skills.Crafting
if skill then
    local skillExp = skill.Exp
    local skillLevel = skill.Level
end
```

### Character Setters

```lua
character.setJob("newJob", flag)  -- flag=true suppresses onJobChange event
character.setJobGrade(grade, flag)
character.setGroup("admin", flag)
character.addMoney(amount)
character.deductMoney(amount)
character.addGold(amount)
character.deductGold(amount)
character.addXp(amount)
character.setRol(amount)
character.setInvCapacity(amount)
character.setDead(boolean)
character.setFirstname(name)
character.setLastname(name)
character.setNickname(nick)
character.setDescription(desc)
```

---

## Users

### getUser (Server)

```lua
local user = Core.getUser(source)
if user then
    -- user available
end
```

### User Methods

```lua
local character = user.getUsedCharacter   -- active character
local characters = user.getCharacters     -- all user characters
local identifier = user.getIdentifier     -- account identifier
```

---

## Callbacks

### Trigger Callback (Server -> Client)

```lua
Core.Callback.Register("myScript:getData", function(source, cb, arg1, arg2)
    local result = { some = "data" }
    cb(result)
end)
```

### Invoke Callback (Client -> Server)

```lua
Core.Callback.Trigger("myScript:getData", function(response)
    print(response.some)
end, arg1, arg2)
```

---

## Webhooks

```lua
-- Send Discord webhook
Core.Webhooks.sendWebhook(
    "webhookUrl",
    "Webhook Title",
    "Message body content",
    16711680,  -- color (decimal RGB)
    {
        { label = "Player", value = GetPlayerName(source) },
        { label = "Action", value = "Something happened" }
    },
    "Avatar URL",
    "Bot Name"
)
```

---

## Instancing

```lua
-- Set player instance (0 = default world)
Core.Instancing.setPlayerInstance(source, instanceId)

-- Get player instance
local instance = Core.Instancing.getPlayerInstance(source)

-- Remove from instance
Core.Instancing.removePlayerInstance(source)
```

---

## Whitelist

```lua
-- Check if player is whitelisted
local isWhitelisted = Core.Whitelist.isWhitelisted(identifier)

-- Add to whitelist
Core.Whitelist.addWhitelist(identifier)

-- Remove from whitelist
Core.Whitelist.removeWhitelist(identifier)
```

---

## Core Events

### onGroupChange

```lua
AddEventHandler("vorp:SelectedCharacter", function(source, character)
    -- Initial setup after character selection
end)
```

### onJobChange

```lua
AddEventHandler("vorp:onJobChange", function(source, newJob, oldJob)
    -- Job changed
end)
```

### onJobGradeChange

```lua
AddEventHandler("vorp:onJobGradeChange", function(source, newGrade, oldGrade, job)
    -- Grade changed
end)
```

### onGroupChange (permission)

```lua
AddEventHandler("vorp:onGroupChange", function(source, newGroup, oldGroup)
    -- Group changed
end)
```

### onPlayerDeath

```lua
AddEventHandler("vorp:onPlayerDeath", function(source, cause, killer)
    -- Player died
end)
```

### Player Spawned

```lua
RegisterNetEvent("vorp:playerSpawned", function()
    -- Client-side: player spawned in world
end)
```

### Revive / Respawn / Heal

```lua
RegisterNetEvent("vorp:revivePlayer", function() end)
RegisterNetEvent("vorp:respawnPlayer", function() end)
RegisterNetEvent("vorp:healPlayer", function() end)
RegisterNetEvent("vorp:onPlayerLevelUp", function(oldLevel, newLevel) end)
```

---

## VORP Lib (Module System)

VORP Lib is a modular library with instance-based components and automatic cleanup on resource restart.

### Setup

Add to `fxmanifest.lua`:

```lua
shared_script "@vorp_lib/import.lua"
```

### Importing Modules

```lua
-- Single module
local Prompts = Import("prompts").Prompts

-- Multiple modules
local modules = Import({"prompts", "inputs", "blips"})
local Prompts = modules.Prompts
local Inputs  = modules.Inputs
local Blips   = modules.Blips

-- Internal file import (from your script)
local helper = Import "/client/utils"

-- External import (from another script)
local shared = Import "@other_script/shared/config"
```

### Available Modules

| Module | Side | Description |
|---|---|---|
| entities | client | Create peds, vehicles, objects |
| blips / map | client | Create blips and map elements |
| inputs | client | Keyboard input controls (Press/Hold/Release) |
| raycast | client | Gameplay raycasts from camera/entity |
| prompts | client | Prompt (interact) UI elements |
| commands | client | Client command registration |
| points | client | Enter/exit point detectors with debug |
| polyzones | client | Polygon, circle, box detection zones |
| events | client | Game event handlers |
| dataview | shared | DataView (binary reader) in Lua |
| streaming | client | Load anim dicts, models, assets |
| commands | server | Server commands with permissions |
| class | shared | OOP class system with inheritance |
| functions | shared | switch, setInterval, etc. |
| logger | shared | Formatted logs with time/level/context |

---

### Entities Module

Creates instanced entities that auto-delete on resource stop.

```lua
local Entity = Import 'entities'

-- Create a Ped
local ped = Entity.Ped:Create({
    Model = 'A_C_COW',
    Pos = vector4(0, 0, 0, 0),
    IsNetworked = true,
    Options = {
        PlaceOnGround = true,
        OutfitPreset = 0,
    },
    OnCreate = function(self)
        print('Ped created:', self:GetHandle())
    end,
    OnDelete = function(handle, netid)
        print('Ped deleted:', handle, netid)
    end
})

-- Create a Vehicle / Wagon
local vehicle = Entity.Vehicle:Create({
    Model = 'wagon01x',
    Pos = vector4(0, 0, 0, 0),
    IsNetworked = true,
    Options = {
        PlaceOnGround = true,
        Seat = { Ped = ped, Index = -1 }
    },
    OnCreate = function(self)
        print('Vehicle:', self:GetHandle())
    end
})

-- Create an Object
local object = Entity.Object:Create({
    Model = 'prop_paper_bag_01',
    Pos = vector4(0, 0, 0, 0),
    IsNetworked = true,
    Options = {
        PlaceOnGround = true,
        Rot = { Pos = vector3(0, 0, 0), Order = 2 }
    }
})

-- Base methods (all entities)
local handle   = ped:GetHandle()
local model    = ped:GetModel()
local pos      = ped:GetPosition()
local heading  = ped:GetHeading()
local rotation = ped:GetRotation()
local netId    = ped:GetNetId()

ped:SetPosition(vector3(10, 10, 0))
ped:Delete()
```

---

### Blips Module

```lua
local Map = Import 'blips'

local blip = Map.Blips:Create('radius', {
    Entity = ped,
    Pos = vector3(2865.88, 475.38, 66.09),
    Radius = 50.0,
    P7 = 0,
    Blip = 1673015813,
    Scale = vector3(1, 1, 1),
    Options = {
        sprite = 1,
        name = 'Test',
        modifier = 'BLIP_MODIFIER_MP_COLOR_1',
        color = 'blue',
    },
    OnCreate = function(self)
        local blue, red, yellow = self:GetBlipColor({'blue', 'red', 'yellow'})
        self:AddModifier(red)
    end
})

local handle = blip:GetHandle()
blip:SetName("New Name")
blip:SetCoords(vector3(0, 0, 0))
blip:SetStyle(1)
blip:SetSprite(1673015813)
blip:AddModifier('BLIP_MODIFIER_MP_COLOR_1')
blip:RemoveModifier('BLIP_MODIFIER_MP_COLOR_1')
blip:AddModifierColor('red')
blip:Remove()
```

**BlipTypes:** `entity`, `coords`, `area`, `radius`

---

### Inputs Module

```lua
local controls = Import 'inputs'

local input = controls.Inputs:Register({
    { inputType = "Press",   key = "E" },
    { inputType = "Hold",    key = "W" },
    { inputType = "Release", key = "S" },
}, function(input, customParams)
    if input.key == "E" then
        print("E pressed")
    elseif input.key == "W" then
        print("W held")
    elseif input.key == "S" then
        print("S released")
    end
end, true)  -- autoStart

input:Pause()
input:Resume()
input:Update({ key = "E", customParam = "value" })
input:RemoveKey("W")
input:Destroy()
```

**inputType:** `Press`, `Hold`, `Release`

---

### Prompts Module

```lua
local PromptModule = Import 'prompts'
local Prompts = PromptModule.Prompts

local prompt = Prompts:Register({
    text = "Interact",
    key = 0xE832C8BB,  -- INPUT_KEY_E hash
    holdTime = 1000,
    group = 0,
    visible = true,
    enabled = true,
    showControl = true,
    onHold = function()
        -- Fires during hold
    end,
    onPressed = function()
        -- Fires on press complete
        print("Prompt activated")
    end
})

prompt:SetVisible(true)
prompt:SetEnabled(true)
prompt:SetText("New Text")
prompt:Delete()
```

---

### Points Module

Enter/exit detection with debug visualization.

```lua
local PointsModule = Import 'points'
local Points = PointsModule.Points

local point = Points:Create({
    coords = vector3(0, 0, 0),
    distance = 3.0,
    debug = true,
    onEnter = function()
        print("Entered point")
    end,
    onExit = function()
        print("Exited point")
    end
})

point:Remove()
```

---

### PolyZones Module

```lua
local ZoneModule = Import 'polyzones'
local Zones = ZoneModule.Zones

-- Circle zone
local circle = Zones.Circle:Create({
    coords = vector3(0, 0, 0),
    radius = 5.0,
    debug = true,
    onEnter = function() print("Entered circle") end,
    onExit  = function() print("Exited circle")  end,
    inside  = function() print("Inside circle")   end,
})

-- Box zone
local box = Zones.Box:Create({
    coords = vector3(0, 0, 0),
    length = 10.0,
    width = 10.0,
    heading = 0.0,
    debug = true,
    onEnter = function() end,
    onExit  = function() end,
    inside  = function() end,
})

-- Poly zone (polygon from points)
local poly = Zones.Poly:Create({
    points = {
        vector2(0, 0), vector2(10, 0),
        vector2(10, 10), vector2(0, 10)
    },
    zMin = 0.0,
    zMax = 10.0,
    debug = true,
    onEnter = function() end,
    onExit  = function() end,
    inside  = function() end,
})

circle:Destroy()
box:Destroy()
poly:Destroy()
```

---

### Streaming Module

Helper for async asset loading.

```lua
local Streaming = Import 'streaming'

-- Load model
local model = joaat('A_C_COW')
Streaming.RequestModel(model, function()
    -- Model loaded
    local ped = CreatePed(model, 0, 0, 0, 0, true, false)
    SetModelAsNoLongerNeeded(model)
end)

-- Load animation dict
Streaming.RequestAnimDict('script_common@shared_scenarios@cutscenes@universal@ig_1@variant_a@', function(dict)
    TaskPlayAnim(ped, dict, 'base', 8.0, -8.0, -1, 0, 0, false, false, false)
end)

-- Load texture dict
Streaming.RequestStreamedTextureDict('blips', function(dict)
    -- ready
end)
```

---

### Logger Module

```lua
local Logger = Import 'logger'

local log = Logger:Create({
    prefix = "MyScript",
    level = "debug",  -- debug, info, warn, error
})

log:debug("Debug message", { extra = "data" })
log:info("Info message")
log:warn("Warning")
log:error("Error occurred")
```

---

## Character Module (Exports)

### Character Events

```lua
-- Client: character selected
RegisterNetEvent("vorp:SelectedCharacter", function(charid)
    print("Character ID:", charid)
end)

-- Server: character selected
AddEventHandler("vorp:SelectedCharacter", function(source, character)
    print("Character:", json.encode(character))
end)

-- Client: new character created
AddEventHandler("vorp:initNewCharacter", function()
    print("New character created")
end)

-- Client: pause character loading scene
TriggerEvent("vorpcharacter:stopLoadingScene", true)  -- pause
TriggerEvent("vorpcharacter:stopLoadingScene", false) -- resume
```

### Character Exports

```lua
-- Client: get all cached components
local comps = exports.vorp_character:GetAllPlayerComponents()

-- Client: get specific component category
local component = exports.vorp_character:GetPlayerComponent("face")

-- Server: open outfits menu
local success = exports.vorp_character:OpenOutfitsMenu(source)
```

### Character StateBags

```lua
-- Bandana worn on face
if LocalPlayer.state.IsBandanaOn then
    -- bandana is on
end

-- Player is in character shop (barber, etc.)
if LocalPlayer.state.PlayerIsInCharacterShops then
    -- in shop
end
```

---

## Inputs Menu (vorp_inputs)

Advanced text/number input dialogs.

```lua
local myInput = {
    type = "enableinput",
    inputType = "input",
    button = "Confirm",
    placeholder = "NAME QUANTITY",
    style = "block",
    attributes = {
        inputHeader = "GIVE ITEM",
        type = "text",                   -- text, number, date, textarea
        pattern = "[0-9]",               -- regex validation
        title = "numbers only",          -- validation tooltip
        style = "border-radius: 10px; background-color: ; border:none;"
    }
}

local result = exports.vorp_inputs:advancedInput(myInput)

-- Split multi-word result
local splitString = {}
for i in string.gmatch(result, "%S+") do
    splitString[#splitString + 1] = i
end
local data1, data2 = splitString[1], splitString[2]
```
