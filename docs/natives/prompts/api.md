---
title: Prompts API
category: Natives
tags:
  - prompts
  - api
  - interaction
framework:
  - redm
difficulty: advanced
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://natives.lawless-street.fr/
---

# Prompts API

Signatures + notes. `Prompt` = integer handle (fwScriptGuid index).

## Register / Delete

### PromptRegisterBegin
```
void PROMPT_REGISTER_BEGIN();
// 0x04F97DE45A519419
// ns HUD
```
Begins a transaction. Every call must be paired with exactly one `PromptRegisterEnd`.

### PromptRegisterEnd
```
Prompt PROMPT_REGISTER_END();
// 0x4F0C395B44A161D5
// ns HUD
```
Commits the transaction, returns the handle for later Set calls.

### PromptDelete
```
void PROMPT_DELETE(Prompt* promptHandle);
// 0x7D629607B68A2A3D
// ns HUD
```
Releases the prompt. Always pair with `PromptRegisterEnd` in `onResourceStop`.

---

## Control / Text

### PromptSetControlAction
```
void PROMPT_SET_CONTROL_ACTION(Prompt prompt, Hash controlHash);
// 0xE532F127887C0958
// ns HUD
```
Binds a prompt to one input (e.g. `` `INPUT_CONTEXT` `` for E). **Call during register transaction (prompt = 0) or after on the returned handle.**

### PromptSetControlActionArray
```
void PROMPT_SET_CONTROL_ACTION_ARRAY(Prompt prompt, Hash* controlHashes, int count);
// 0x26538853A732C0A5
// ns HUD
```
Binds multiple alternate keys to the same prompt.

### PromptSetText
```
void PROMPT_SET_TEXT(Prompt prompt, char* textVarString);
// 0x4419980E24D783AC
// ns HUD
```
Sets prompt label. Pass the handle or `0` during reg.

The canonical way to pass a raw string is via `CreateVarString`, as in:

```lua
PromptSetText(0, CreateVarString(10, "LITERAL_STRING", "Open"))
```

### PromptSetTextLabel
```
void PROMPT_SET_TEXT_LABEL(Prompt prompt, char* labelName);
// 0xD4BB2419F6E13EDF
// ns HUD
```
Uses a registered `AddTextEntry` label key, avoiding the `CreateVarString` dance for localised text.

---

## Mode / Time

### PromptSetHoldMode
```
void PROMPT_SET_HOLD_MODE(Prompt prompt, BOOL toggle);
// 0xB446788218282E59
// ns HUD
```
Turns the prompt into "hold-to-activate". Combine with `PromptSetHoldTime`.

### PromptSetHoldTime
```
void PROMPT_SET_HOLD_TIME(Prompt prompt, int holdTimeMs);
// 0xCA510B53F4F910F2
// ns HUD
```
Hold duration in milliseconds (default is 1000).

### PromptHasHoldModeCompleted
```
BOOL PROMPT_HAS_HOLD_MODE_COMPLETED(Prompt prompt);
// 0xE2581ADF0010E480
// ns HUD
```
Poll per-frame inside your interaction check. It returns true **for exactly one frame** at completion.

### PromptSetStandardMode
```
void PROMPT_SET_STANDARD_MODE(Prompt prompt, BOOL toggle);
// 0x5B346F561C4EE970
// ns HUD
```
"Standard" = press-and-release mode (default). Use this for non-hold prompts so the engine styles the hint correctly.

### PromptSetEnabled
```
void PROMPT_SET_ENABLED(Prompt prompt, BOOL enabled);
// 0x03B940F3C6FA108C
// ns HUD
```
If false, prompt is greyed out but still takes up layout space.

### PromptSetVisible
```
void PROMPT_SET_VISIBLE(Prompt prompt, BOOL visible);
// 0x71215AC8CF73306A
// ns HUD
```
If false, prompt is removed entirely from the HUD (no gap).

---

## Prompt Groups

Use groups to render multiple prompts as a cohesive unit (e.g. same control, same on-screen anchor point).

### PromptSetGroup
```
void PROMPT_SET_GROUP(Prompt prompt, int groupId);
// 0xA7543D6A082D11D7
// ns HUD
```
`groupId` is any integer you pick (typically `0` for "no group", `1` … `N` per interaction point).

### UiPromptSetActiveGroupThisFrame
```
void UI_PROMPT_SET_ACTIVE_GROUP_THIS_FRAME(int groupId, BOOL force);
// 0x817E55D33CA63CCB
// ns HUD
```
**Must be called every frame you want the group to show**, even if each prompt's `SetVisible` is true. This is the single most-missed call; without it, grouped prompts never render.

Typical pattern:
```lua
CreateThread(function()
    while true do
        Wait(0)
        local dist = #(pos - GetEntityCoords(PlayerPedId()))
        if dist < 2.0 then
            UiPromptSetActiveGroupThisFrame(SHOP_GROUP, true)
            PromptSetEnabled(lootPrompt, true)
            PromptSetVisible(lootPrompt, true)
            PromptSetEnabled(hidePrompt, true)
            PromptSetVisible(hidePrompt, true)
        else
            PromptSetEnabled(lootPrompt, false)
            PromptSetVisible(lootPrompt, false)
            PromptSetEnabled(hidePrompt, false)
            PromptSetVisible(hidePrompt, false)
        end
    end
end)
```

### UiPromptHasAnyPromptJustBecomePressedThisFrame
```
BOOL UI_PROMPT_HAS_ANY_PROMPT_JUST_BECOME_PRESSED_THIS_FRAME();
// 0x11ED239D5FF51B53
// ns HUD
```
Useful inside a group to short-circuit: if any prompt in any group was just pressed, skip polling this tick.

---

## Priority / Layout

### PromptSetUiPriority
```
void PROMPT_SET_UI_PRIORITY(Prompt prompt, int priority);
// 0x51748CF64B1B510F
// ns HUD
```
Higher = renders first in the group. Use `0` for primary (E), `1` for secondary (R), etc.

### PromptSetBackgroundColor
```
void PROMPT_SET_BACKGROUND_COLOR(Prompt prompt, int color);
// 0x7DA656B8
// ns HUD
```
`color` is an `UIFEED_COLOR_*` integer (0 = default, 1 = white, 2 = red, 6 = green, 11 = blue, 19 = orange).

### PromptSetInputMeterFillColor
```
void PROMPT_SET_INPUT_METER_FILL_COLOR(Prompt prompt, int color);
// 0x91C16B41
// ns HUD
```
Colour of the hold-mode progress ring. Same `UIFEED_COLOR_*` integers as background.

---

## Per-Frame Polling Helpers

These complement prompt handling (not prompt natives per-se, but you always use them together):

```lua
-- Was key released this frame? (press-mode prompt)
IsControlJustReleased(0, `INPUT_CONTEXT`)
-- Was key just pressed?
IsControlJustPressed(0, `INPUT_CONTEXT`)
-- Is key currently held?
IsControlPressed(0, `INPUT_CONTEXT`)
-- Disable a control temporarily (so E doesn't also trigger the default)
DisableControlAction(0, `INPUT_CONTEXT`, true)
```

Tip: `DisableControlAction` + custom prompt = prevents the default "interact with pickup" R* input from also firing while your script consumes that key for a custom prompt.
