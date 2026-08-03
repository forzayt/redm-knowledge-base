---
title: Entities
category: Natives
tags:
  - entities
  - network
  - world-state
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://natives.lawless-street.fr/
  - https://alloc8or.re/rdr3/nativedb/?n=CreateObject
  - https://docs.fivem.net/docs/scripting-manual/entity-creation-and-ownership/
---

# Entities

In RDR3, every world object, ped, vehicle (wagon), horse, weapon pickup, and placed prop is an **entity**. Natives let you create them, position them, attach them, make them networked, and destroy them safely.

The 3 entity classes are:

| Kind | Creation native | Deletion native |
|---|---|---|
| `object` (prop / scene prop) | `CreateObject(model, x, y, z, isNet, mission, doorFlags)` | `DeleteObject(obj)` + `SetEntityAsNoLongerNeeded` |
| `ped` (human, animal, MP) | `CreatePed(type, model, x, y, z, heading, isNet, pinned)` | `DeletePed(ped)` + `SetEntityAsNoLongerNeeded` |
| `vehicle` (wagon, cart, boat, train car) | `CreateVehicle(model, x, y, z, heading, isNet, pinned)` | `DeleteVehicle(v)` + `SetEntityAsNoLongerNeeded` |

All three respond to the shared `ENTITY` namespace natives: `GetEntityCoords`, `SetEntityCoords`, `GetEntityHeading`, `FreezeEntityPosition`, `SetEntityInvincible`, `DoesEntityExist`, `NetworkGetNetworkIdFromEntity`, etc.

## Local vs Networked

Every entity is either **local** (only visible to the script that created it, inside one client) or **networked** (replicated through the server to other clients via state sync).

- **For props/NPCs a single player interacts with, e.g. a local crafting bench:** Create locally with `isNetwork = false`. No server round-trip; no ownership questions; despawns if creator leaves.
- **For persistent / shared items — dropped items, stable horses, locked doors:** Create with `isNetwork = true` and store/use `NetworkGetNetworkIdFromEntity(ent)`. Any client can translate the net ID back via `NetToObj(netId)`.

### Ownership (Networking)

The server assigns networked entities an "owner" client — normally the spawner. Only the owner can move the entity freely; other clients see a ghosted copy that drifts back to owner-authored state via 1Hz sync. To forcibly reassign ownership:

- Server: `NetworkRequestControlOfEntity(entity)` or `NetworkMigrateEntityOwnership(entity)` (CFX).
- Never move a networked entity from a non-owner client; queue the intent as a server event and have the owner apply it.

## Cleanup — Always Pair Create + Delete

```lua
local created = {}

AddEventHandler("onResourceStop", function(res)
    if res ~= GetCurrentResourceName() then return end
    for _, e in ipairs(created) do
        if DoesEntityExist(e) then
            DeleteEntity(e)
            SetEntityAsNoLongerNeeded(e)
        end
    end
end)

local function createProp(model, coords, net)
    RequestModel(model)
    while not HasModelLoaded(model) do Wait(0) end
    local prop = CreateObject(model, coords.x, coords.y, coords.z, net, false, false)
    SetModelAsNoLongerNeeded(model)
    PlaceObjectOnGroundProperly(prop)
    FreezeEntityPosition(prop, true)
    table.insert(created, prop)
    return prop
end
```

## See Also

- [entities/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/entities/index.md)
- [entities/api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/entities/api.md) — signatures for creation, deletion, attachment, state, networking helpers
- [entities/examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/entities/examples.md) — create/freeze prop, attach to ped, wagon with NPC driver pattern
