---
title: Prompts
category: Natives
tags:
  - prompts
  - ui
  - interaction
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://natives.lawless-street.fr/
  - https://github.com/femga/rdr3_discoveries
  - https://alloc8or.re/rdr3/nativedb/?n=PromptRegisterBegin
---

# Prompts

Prompts are the native RDR2 "Press [E] to interact" UI elements displayed in the bottom-right of the screen. They are the engine's answer to FiveM's `ESX.ShowHelpNotification`, but far more powerful: hold-to-activate, grouping, control-aware text, visibility modes, and priority ordering.

## Control Hashes

Most scripts use these; hash via `` ` `` literal (or `GetHashKey` on the AllControlActions enum):

| Key | Hash (literal) |
|---|---|
| `E` | `INPUT_CONTEXT` → `0xCEFD9220` |
| `G` | `INPUT_INTERACT_ANIMAL` → `0x760A9C6F` |
| `R` | `INPUT_RELOAD` → `0xE30CD707` |
| `H` | `INPUT_HORSE_STOW_WEAPON` → `0x24978A28` |
| `F` | `INPUT_ENTER` → `0xB2F377E8` |
| `Q` | `INPUT_CREATOR_LS` → `0xDE794E3E` |
| `TAB` | `INPUT_WEAPON_WHEEL` → `0xB238FE0B` |

## Life-cycle

A prompt is a handle returned by `PromptRegisterEnd` and must be:

1. Allocated with `PromptRegisterBegin`
2. Configured (control, text, hold time, groups, visibility, etc.)
3. Committed with `PromptRegisterEnd() → handle`
4. Toggled each frame with `PromptSetEnabled(h, true/false)` / `PromptSetVisible(h, true/false)`
5. Deleted with `PromptDelete(h)` on `onResourceStop`

Example of a single E prompt:

```lua
local p = nil
do
    PromptRegisterBegin()
    PromptSetControlAction(0, `INPUT_CONTEXT`)
    PromptSetText(CreateVarString(10, "LITERAL_STRING", "Open Door"))
    PromptSetEnabled(0, true)
    PromptSetVisible(0, true)
    PromptSetHoldMode(0, true)              -- or hold time ms via PromptSetHoldTime()
    PromptSetStandardMode(0, true)
    p = PromptRegisterEnd()
end

CreateThread(function()
    local pos = vector3(0, 0, 0)
    while true do
        Wait(0)
        local near = #(pos - GetEntityCoords(PlayerPedId())) < 2.0
        PromptSetEnabled(p, near)
        PromptSetVisible(p, near)
        if near then
            if PromptHasHoldModeCompleted(p) then
                print("Door opened")
                break
            end
        end
    end
end)

AddEventHandler("onResourceStop", function(r)
    if r == GetCurrentResourceName() then
        PromptDelete(p)
    end
end)
```

## See Also

- [prompts/index.md](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/index.md)
- [prompts/api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/api.md) — full native list: create/delete, text, hold mode, groups, priority, attributes
- [prompts/examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/examples.md) — grouped prompts, hold-vs-press, distance-gated multi-action prompt
