---
title: Prompts Index
category: Natives
tags:
  - prompts
  - interaction
  - ui
framework:
  - redm
difficulty: intermediate
last_updated: 2026-08-03
sources:
  - https://docs.redm.store/natives
  - https://alloc8or.re/rdr3/nativedb/
  - https://github.com/femga/rdr3_discoveries
---

# Prompts

Prompts are the bottom-right "Press [E] to do X" widget built into RDR2's HUD. The engine uses a register-and-then-poll model: you **register** each prompt once at startup (not per tick!) producing a `prompt handle`, and then every tick you **set visibility / enabled state / text** on that handle based on current game state.

> Community wrappers exist (VORP Lib `prompts`, BLN Lib `Prompt`, `redm-native` exports) but for anything non-trivial it helps to understand the native life-cycle — which is what these pages cover.

## Sub-pages

| File | Content |
|---|---|
| [api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/api.md) | Per-native signatures for register/configure/set/delete helpers, plus prompt groups and attributes |
| [examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/examples.md) | Recipes: single E-key prompt, hold 1.5s, grouped Loot/Steal/Leave, distance-gated shop interaction |

## Life-cycle in 4 lines

```lua
PromptRegisterBegin()                              -- 1. start a new prompt descriptor
PromptSetControlAction(0, `INPUT_CONTEXT`)         -- 2. configure: which key, text, mode, etc
PromptSetText(CreateVarString(10, "LITERAL_STRING", "Pick up"))
PromptSetEnabled(0, true)
local handle = PromptRegisterEnd()                 -- 3. commit -> integer handle

-- 4. each tick:  PromptSetEnabled(handle, near) ; PromptSetVisible(handle, near)
```

### Register-time vs Per-tick calls

| Register-time (once) | Per-tick (every 0 ms) |
|---|---|
| `PromptRegisterBegin` / `PromptRegisterEnd` | `PromptSetEnabled` |
| `PromptSetControlAction` | `PromptSetVisible` |
| `PromptSetText` (or use `PromptSetTextLabel`) | `PromptSetText` (if dynamic) |
| `PromptSetGroup` | `PromptSetGroup` (if moving between groups) |
| `PromptSetHoldMode` / `PromptSetHoldTime` | `PromptHasHoldModeCompleted` |
| `PromptSetStandardMode` | `PromptIsJustPressed` |
| `PromptSetUiPriority` | — |
| `PromptSetControlActionArray` (multiple keys) | — |

**Rule:** If you call `PromptRegisterBegin` more than once for the same interaction, you're leaking handles. Wrap registration once and store the handle.

---

## Control Hashes

Use `` `INPUT_*` `` backtick literals (pre-computed hash). These are the most-used ones:

| Literal | Key |
|---|---|
| `INPUT_CONTEXT` | `E` (0xCEFD9220) |
| `INPUT_INTERACT_ANIMAL` | `G` (0x760A9C6F) |
| `INPUT_RELOAD` | `R` (0xE30CD707) |
| `INPUT_HORSE_STOW_WEAPON` | `H` (0x24978A28) |
| `INPUT_ENTER` | `F` (0xB2F377E8) |
| `INPUT_CREATOR_LS` | `Q` (0xDE794E3E) |
| `INPUT_WEAPON_WHEEL` | `TAB` (0xB238FE0B) |
| `INPUT_DUCK` | `CTRL` (0xDB096B85) |
| `INPUT_SPRINT` | `SHIFT` (0x8FFC75D6) |
| `INPUT_JUMP` | `SPACE` (0xD9D0E1C0) |
| `INPUT_FRONTEND_ACCEPT` | `ENTER` (0xC7B5340A) |
| `INPUT_MELEE_ATTACK_LIGHT` | `MouseLeft` (0x07CE1E61) |
| `INPUT_AIM` | `MouseRight` (0xF84FA74F) |
| `INPUT_SELECT_WEAPON_KNIFE` | `1` (0xE6F612E4) |

For the full list, see femga's `controls` discovery repo or use VORP Lib `inputs` module which wraps all of them.

## Three Activation Modes

### 1. Press (instant)
```lua
PromptSetStandardMode(0, true)            -- register-time
-- Per tick check:
if IsControlJustReleased(0, `INPUT_CONTEXT`) then ... end
```

### 2. Hold (hold to complete)
```lua
PromptSetHoldMode(0, true)                -- register-time
PromptSetHoldTime(0, 1500)                -- 1.5s
-- Per tick check:
if PromptHasHoldModeCompleted(handle) then ... end
```

### 3. Grouped (multiple prompts sharing one E/G/F display)
See `PromptSetGroup` in [api.md](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/api.md) and [examples.md](file:///d:/Playground/redm-knowledge-base/docs/natives/prompts/examples.md).

## Best Practices

1. **Register once per prompt handle, ever.** Never register inside a `Wait(0)` loop.
2. **Group by activity, not by key.** For a corpse with `Loot [E]` and `Hide [R]` — use the same group.
3. **Favour HoldMode for destructive actions** (burn, steal, kill) to avoid accidental taps.
4. **Keep text short.** ≤ 16 chars is safe without clipping on 720p displays.
5. **Always delete on `onResourceStop`.** Leaked prompt handles accumulate and you'll eventually get zero new prompts rendering until restart.
