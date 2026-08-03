---
title: Entities API
category: Natives
tags:
  - entities
  - api
  - world-state
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://natives.lawless-street.fr/
---

# Entities API

Signatures below use CFX aliases where they exist; otherwise fall back to `Citizen.InvokeNative(0xHASH, …)`.

## Creation

### CreateObject
```
Object CREATE_OBJECT(Hash modelHash, float x, float y, float z, BOOL isNetwork, BOOL netMissionEntity, BOOL doorFlag);
// 0x501BD02B7D07336B (CREATE_OBJECT_NO_OFFSET: 0x9840B1498C12070B)
// ns OBJECT
```
Spawn an object prop. Use `CreateObjectNoOffset` when you don't want engine-level "smart placement" corrections for a door/shutter.

- `modelHash`: `` `hash_literal` ``. Must be loaded with `RequestModel` first.
- `x,y,z`: spawn position (world coords).
- `isNetwork`: `true` to replicate through the server to all clients on the scope.
- `netMissionEntity`: `true` pins the entity for the server (forces `SetEntityAsMissionEntity` semantics).
- `doorFlag`: `false` for standard props; `true` if the prop is a door/shutter so the engine attaches door physics.

### CreateVehicle
```
Vehicle CREATE_VEHICLE(Hash modelHash, float x, float y, float z, float heading, BOOL isNetwork, BOOL bScriptHostPed);
// 0xAF35D0D2583051B0
// ns VEHICLE
```
Spawn a wagon/cart/canoe/train vehicle.

### CreateVehicleServerSetter (Server)
```
void CREATE_VEHICLE_SERVER_SETTER(Hash modelHash, int vehicleType, float x, float y, float z, float heading);
// 0xE4E13355E80884FC
// ns VEHICLE (server-only CFX)
```
Server-side vehicle creation — preferred over client creation for persistent/shared wagons because the server is the first owner.

### CreatePedInsideVehicle
```
Ped CREATE_PED_INSIDE_VEHICLE(Vehicle vehicle, int pedType, Hash modelHash, int seatIndex, BOOL isNetwork, BOOL bScriptHostPed);
// 0x3000F092
// ns PED
```

---

## Deletion

### DeleteEntity / DeleteObject / DeletePed / DeleteVehicle
```
void DELETE_ENTITY(Entity* entity);       // 0xFAA0C035AA2B6A9F
void DELETE_OBJECT(Object* object);       // 0x539E0AE3E6634B9F
void DELETE_PED(Ped* ped);                // 0x96145A2843EEF7B4
void DELETE_VEHICLE(Vehicle* vehicle);    // 0x4A8EEF7C379A77D0
// ns ENTITY / OBJECT / PED / VEHICLE
```
All variants take the handle **by pointer**; Lua's binding handles this transparently.

**Pattern:**
```lua
if DoesEntityExist(ent) then
    SetEntityAsMissionEntity(ent, false, true)
    DeleteEntity(ent)
end
```

### SetEntityAsNoLongerNeeded
```
void SET_ENTITY_AS_NO_LONGER_NEEDED(Entity* entity);
// 0x2B9D688EA7D2AA1F
// ns ENTITY
```
Releases the engine's mission-entity refcount. Engine is then free to garbage-collect the entity if the player leaves the streaming cell. Always call after Delete.

### SetEntityAsMissionEntity
```
BOOL SET_ENTITY_AS_MISSION_ENTITY(Entity entity, BOOL p1, BOOL p2);
// 0x071EF1C83E204D1E
// ns ENTITY
```
Use `(ent, true, false)` right after creation to mark "this script owns this entity; don't cull it."

---

## Position / Rotation

### GetEntityCoords
```
Vector3 GET_ENTITY_COORDS(Entity entity, BOOL alive);
// 0x1647F1CB7EE70464
// ns ENTITY
```

### SetEntityCoords / SetEntityCoordsNoOffset
```
void SET_ENTITY_COORDS(Entity ent, float x, float y, float z, BOOL alive, BOOL deadFlag, BOOL ragdollFlag, BOOL clearArea);
// 0x621873CE5A8960DD
```
Teleport entity. Use `NoOffset` to skip engine "smart z-adjustment" for doors/interiors.

### GetEntityHeading / SetEntityHeading
```
float GET_ENTITY_HEADING(Entity entity);        // 0x972CCF873B83AF17
void SET_ENTITY_HEADING(Entity ent, float h);   // 0x7C0B6080181892CB
```
Heading is in degrees (0 = north, 90 = east, 180 = south, 270 = west).

### SetEntityRotation
```
void SET_ENTITY_ROTATION(Entity ent, float pitch, float roll, float yaw, int rotationOrder, BOOL p5);
// 0x8FF4EAB07F2C0D0D
```
Use `rotationOrder=2` for standard YXZ "pitch/roll/yaw" Euler (most intuitive for placing crates, barrels, signs).

---

## Visibility / Collision / Invincibility

```
void FREEZE_ENTITY_POSITION(Entity ent, BOOL freeze);                  // 0x428CA6DBD1094446
void SET_ENTITY_VISIBLE(Entity ent, BOOL visible, BOOL unk);          // 0xEA1C6FA1CE917E2C
void SET_ENTITY_INVINCIBLE(Entity ent, BOOL invincible);             // 0x600C1212CE3F1E6F
void SET_ENTITY_COLLISION(Entity ent, BOOL hasColl, BOOL keepPhysics); // 0x1A9205D63D0C1BAB
void SET_ENTITY_COMPLETELY_DISABLE_COLLISION(Entity ent, BOOL flag, BOOL ignore);
// 0x1199F557D272E01D
BOOL DOES_ENTITY_EXIST(Entity entity);                                 // 0x79911642455C6D4C
Hash GET_ENTITY_MODEL(Entity ent);                                     // 0x9F47B058362C83DB
```

---

## Attach / Detach

### AttachEntityToEntity
```
void ATTACH_ENTITY_TO_ENTITY(Entity ent1, Entity ent2,
    int boneIndex,
    float offX, float offY, float offZ,
    float rotX, float rotY, float rotZ,
    BOOL p9, BOOL useSoftPinning, BOOL collision, BOOL isPed,
    int rotationOrder, BOOL useFixedRotation);
// 0x4B741B68D17765AB
// ns ENTITY
```
- `boneIndex`: Use `GetPedBoneIndex(ped, bone)` or `0` for world-attachment.
- `offX/Y/Z`: Local offset (centimetres in R* convention — 1.0 = 1 cm).
- `rotX/Y/Z`: Local Euler rotation.
- `collision`: `false` to let the attached prop pass through scenery; `true` is usually buggy.
- `isPed`: `true` when attaching a ped (rare), otherwise `false`.
- `rotationOrder`: `2` for standard YXZ.

### DetachEntity
```
void DETACH_ENTITY(Entity ent, BOOL unkBool, BOOL collision);
// 0x9A6B81408FE91AB1
// ns ENTITY
```

### GetEntityAttachedTo
```
Entity GET_ENTITY_ATTACHED_TO(Entity ent);
// 0x34906F4C57D5291A
// ns ENTITY
```
Returns 0 if not attached.

---

## Networking Helpers

```
int  NETWORK_GET_NETWORK_ID_FROM_ENTITY(Entity ent);        // 0x4DB0EA75C9E03BA5
BOOL NETWORK_DOES_NETWORK_ID_EXIST(int netId);              // 0x8704DFA6EC20BD30
Object NET_TO_OBJ(int netId);                               // 0x9FDD0BD57DE2859B
Ped    NET_TO_PED(int netId);                               // 0x1EBE8D3D3D26D2C5
Vehicle NET_TO_VEH(int netId);                              // 0x6E010B6D5B13917E
BOOL NETWORK_HAS_CONTROL_OF_ENTITY(Entity ent);             // 0x4FC4AA7688CE1516
BOOL NETWORK_REQUEST_CONTROL_OF_ENTITY(Entity ent);         // 0x07315A175169D3AD
```

### Server-only ownership migration (CFX)
```
void NETWORK_MIGRATE_ENTITY_OWNERSHIP(Entity ent, Player newOwner);  // 0xA33B04B1CA0403EB
```

---

## Object-Specific

```
void PLACE_OBJECT_ON_GROUND_PROPERLY(Object obj);            // 0x45B51C62A2A189F1
void SET_OBJECT_TARGETTABLE(Object obj, BOOL targettable);   // 0x2616062DF0CDFC6C
void ACTIVATE_PHYSICS(Object obj);                           // 0x7C3A14E9142FC2DB
```

## Vehicle-Specific

```
void SET_VEHICLE_ON_GROUND_PROPERLY(Vehicle v);              // 0xB2407810987481DD
int  GET_VEHICLE_PED_IN(Ped ped);                            // 0x9A912718
void SET_VEHICLE_DOORS_LOCKED(Vehicle v, int lockStatus);    // 0x2D0BBAE158EFE6BA
// lockStatus: 0 = none, 1 = locked, 2 = locked+player can break, 4 = unlocked
```
