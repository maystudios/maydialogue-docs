---
description: Drive dialogue input through Enhanced Input — keyboard, gamepad and touch, fully remappable.
---

# Enhanced Input Integration

MayDialogue speaks **Enhanced Input** — UE5's modern, data-driven input system. With it you define your dialogue controls (advance, skip, abort, pick a choice) as `UInputAction` assets, make them remappable project-wide, and let the plugin automatically detect whether the player is currently using keyboard or gamepad — the skip-button glyph and the Slate "continue" hint adapt on their own.

You don't strictly have to set anything up: if you turn Enhanced Input off or leave actions unassigned, MayDialogue falls back to hardcoded keys and works immediately.

{% hint style="info" %}
This page supersedes the manual `SetInputDevice` path from the [Skip Button chapter](../ui/skip-button.md) as the recommended approach. `SetInputDevice` remains available as a **manual override** — you can override the device detection yourself at any time.
{% endhint %}

## Ready to run: the fallback keys

With no setup at all, the dialogue already responds to sensible default keys. This binding always applies when Enhanced Input is disabled **or** an action slot is left empty:

| Action | Keyboard | Gamepad |
| --- | --- | --- |
| Advance / skip typewriter | `Space`, `Enter` | `(A)` |
| Choice up | `Arrow ↑`, `W` | `D-Pad ↑` |
| Choice down | `Arrow ↓`, `S` | `D-Pad ↓` |
| Confirm choice | `Enter`, `Space` | `(A)` |
| Pick choice directly | `1` – `9` | — |
| Abort dialogue | `Esc` | `(B)` |

For a game jam or prototype this is entirely enough. If you want remappable input or your own keys, set up Enhanced Input — the next section shows it step by step.

## Step by step: create IA and IMC assets

### Step 1 — Create Input Action assets

Create an `Input Action` asset for each dialogue action (**right-click in the Content Browser → Input → Input Action**). For a complete setup you need:

```text
IA_Dialogue_Advance        (Digital / bool)
IA_Dialogue_Skip           (Digital / bool)
IA_Dialogue_Abort          (Digital / bool)
IA_Dialogue_ChoiceUp       (Digital / bool)
IA_Dialogue_ChoiceDown     (Digital / bool)
IA_Dialogue_ChoiceConfirm  (Digital / bool)
IA_Dialogue_Choice1 … 9    (Digital / bool — optional, for direct pick)
```

All are value-type **Digital (bool)** — they are simple button presses.

### Step 2 — Create a Mapping Context

Create an `Input Mapping Context` (**right-click → Input → Input Mapping Context**), e.g. `IMC_Dialogue`. Add the desired keys per action:

```text
IMC_Dialogue
 ├─ IA_Dialogue_Advance      → Space, Gamepad Face Button Bottom
 ├─ IA_Dialogue_Abort        → Escape, Gamepad Face Button Right
 ├─ IA_Dialogue_ChoiceUp     → Up, W, Gamepad D-Pad Up
 ├─ IA_Dialogue_ChoiceDown   → Down, S, Gamepad D-Pad Down
 ├─ IA_Dialogue_ChoiceConfirm→ Enter, Gamepad Face Button Bottom
 └─ IA_Dialogue_Choice1…9    → 1 … 9
```

> 📸 **Image placeholder:** `enhanced-input-imc-editor.png` — Input Mapping Context editor with the dialogue actions and their key bindings.
> *Setup:* `IMC_Dialogue` opened in the asset editor. Mappings list visible: `IA_Dialogue_Advance` with keys "Space" and "Gamepad Face Button Bottom", below it `IA_Dialogue_Abort` with "Escape" and "Gamepad Face Button Right". At least three actions expanded so the key assignment is clearly visible.

### Step 3 — Assign in Project Settings

Open **Edit → Project Settings → Plugins → MayDialogue**, category **Dialogue | Enhanced Input**, and fill in your assets:

```text
Use Enhanced Input          = true        (default)
Dialogue Mapping Context     = IMC_Dialogue
Mapping Context Priority     = 100         (default)
Auto Detect Input Device     = true        (default)

Advance Action               = IA_Dialogue_Advance
Skip Action                  = IA_Dialogue_Skip
Abort Action                 = IA_Dialogue_Abort
Choice Up Action             = IA_Dialogue_ChoiceUp
Choice Down Action           = IA_Dialogue_ChoiceDown
Choice Confirm Action        = IA_Dialogue_ChoiceConfirm
Choice By Index Actions       = [ IA_Dialogue_Choice1 … IA_Dialogue_Choice9 ]
```

That's it. As soon as a dialogue starts, MayDialogue adds the `DialogueMappingContext` with the given priority and removes it again when the dialogue ends.

> 📸 **Image placeholder:** `enhanced-input-project-settings.png` — Project Settings → Plugins → MayDialogue, category "Dialogue | Enhanced Input" with the Enhanced Input fields filled in.
> *Setup:* Project Settings window, MayDialogue section, scrolled to the Input category. `Use Enhanced Input` checkbox active, `Dialogue Mapping Context` shows `IMC_Dialogue`, the action slots (`Advance Action`, `Abort Action`, …) populated with the IA assets. Red border around the Enhanced Input fields.

## Automatic device detection

When `bAutoDetectInputDevice = true` (default), the plugin watches which input device was used last and updates the UI accordingly:

* The **skip button** shows the matching glyph (spacebar symbol for keyboard, `(A)` for gamepad).
* The **Slate "continue" hint** in the debug widget shows the same device hint.

You don't have to bind anything — detection feeds `SkipButton` and the Slate hint automatically. If the player switches from keyboard to controller mid-conversation, the glyph swaps live.

{% hint style="info" %}
If you need full control (e.g. because your project has its own device detection), set `bAutoDetectInputDevice = false` and call `SetInputDevice(bool bIsGamepad)` on the skip button as before. The manual override then takes precedence.
{% endhint %}

## Remapping recipe: let players rebind keys

Because dialogue control runs through Enhanced Input, you get remapping almost for free. Use UE's `Enhanced Input User Settings` (player-mappable keys):

1. In your `IMC_Dialogue`, mark the mappings the player may change as **Player Mappable** (give each a `Mapping Name`, e.g. `Dialogue_Advance`).
2. Build a settings widget that calls `Map Player Key In Slot` on the `Enhanced Input User Settings` when the player presses a new key.
3. Done — the new binding immediately applies in dialogue too, since MayDialogue reads the same actions.

```text
[Player presses a new key in the options menu]
    │
    ▼
[Map Player Key In Slot]   (Enhanced Input User Settings)
    ├─ Mapping Name: "Dialogue_Advance"
    └─ New Key:      (captured key)
         → from now on the dialogue advances with the new key
```

> 📸 **Image placeholder:** `enhanced-input-remap-menu.png` — In-game options menu with a remappable "Dialogue advance" row.
> *Setup:* Simple settings widget in PIE. Row "Dialogue: Advance" with the current key "Space" and a "Rebind" button. Next to it (or as a second image) the moment after rebinding, the key now showing "E".

## API reference: settings fields

All fields live on `UMayDialogueSettings`, category **Dialogue | Enhanced Input** (except `bAutoDetectInputDevice`, which is in the same category). Every field except `bUseEnhancedInput` has an `EditCondition` on `bUseEnhancedInput`, so they grey out when the master switch is off.

| Field | Type | Default | Purpose |
| --- | --- | --- | --- |
| `bUseEnhancedInput` | `bool` | `true` | Master switch. Off = only the fallback keys apply. |
| `DialogueMappingContext` | `TSoftObjectPtr<UInputMappingContext>` | *(empty)* | IMC added on dialogue start and removed on end. |
| `MappingContextPriority` | `int32` | `100` | Priority the IMC is pushed with (higher = overrides lower contexts). |
| `bAutoDetectInputDevice` | `bool` | `true` | Auto-detects keyboard/gamepad and feeds the skip glyph + Slate hint. |
| `AdvanceAction` | `TSoftObjectPtr<UInputAction>` | *(empty)* | "Advance" / typewriter skip. |
| `SkipAction` | `TSoftObjectPtr<UInputAction>` | *(empty)* | Voice / line skip (if wanted separately from advance). |
| `AbortAction` | `TSoftObjectPtr<UInputAction>` | *(empty)* | Abort the dialogue. |
| `ChoiceUpAction` | `TSoftObjectPtr<UInputAction>` | *(empty)* | Choice cursor up. |
| `ChoiceDownAction` | `TSoftObjectPtr<UInputAction>` | *(empty)* | Choice cursor down. |
| `ChoiceConfirmAction` | `TSoftObjectPtr<UInputAction>` | *(empty)* | Confirm the highlighted choice. |
| `ChoiceByIndexActions` | `TArray<TSoftObjectPtr<UInputAction>>` | *(empty)* | Direct pick: index 0 = choice 1, index 1 = choice 2, … Add up to nine entries (mirrors the hardcoded 1–9 number keys). |

{% hint style="info" %}
All action slots are `TSoftObjectPtr` — empty slots fall back to their fallback key **individually**. So you can, for example, assign only `AdvanceAction` and `AbortAction` and leave choice navigation to the default keys.
{% endhint %}

## Gotchas

| Case | What happens / what to watch for |
| --- | --- |
| **Enhanced Input plugin not enabled** | If the engine-side Enhanced Input plugin is disabled in the project, only the fallback keys apply — regardless of `bUseEnhancedInput`. |
| **IMC set, but actions empty** | The context is pushed, but with no assigned actions nothing dialogue-specific happens → the empty slots fall back to the fallback keys. |
| **`bSwitchToUIInputDuringDialogue` interaction** | During dialogue the plugin switches to Game+UI input mode by default (mouse visible, see [Project Settings](../getting-started/project-settings.md)). Make sure your IMC still receives input in that mode. |
| **Device detection flickers** | Some setups fire phantom inputs (e.g. a connected but unused controller). Set `bAutoDetectInputDevice = false` and drive the glyph manually via `SetInputDevice`. |
| **Direct choice pick beyond 9** | `ChoiceByIndexActions` covers 1–9. Navigate choice lists with more entries via Up/Down/Confirm. |
