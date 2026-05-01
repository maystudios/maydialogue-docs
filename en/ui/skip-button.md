---
description: The skip button — when it is visible, how it responds to different input devices, and how to replace it.
---

# Skip Button

`UMayDialogueWidget_SkipButton` shows the player how to proceed — either by skipping the typewriter or advancing the dialogue. The widget is platform-aware: it automatically displays the correct hint for keyboard, gamepad, or touch.

> 📸 **Image placeholder:** `skip-button-ingame.png` — PIE viewport, dialogue running. Skip button visible in the bottom-right: icon (spacebar symbol for keyboard), text "Continue". Red border around the skip area.
> *Setup:* Start PIE, trigger a dialogue (keyboard input active). Take a screenshot of the skip button in the bottom-right.

## When is the skip button visible?

The skip button is always visible during an active dialogue. The top-level widget controls its visibility — `ShowFrame` on dialogue start, `HideFrame` on dialogue end.

What happens on click is determined by the top-level widget:

```
Click on SkipButton
  → RequestAdvance() in the top-level widget
      → IsTypewriterActive()?
          True  → SkipTypewriter()   ← show text in full immediately
          False → AdvanceDialogue()  ← next line / end dialogue
```

## BindWidget slots

| Name in Designer | Type | Purpose |
|---|---|---|
| `SkipClickButton` | `UButton` | The clickable area |
| `SkipLabel` | `UTextBlock` | Platform hint (populated automatically) |

Both are optional. Without `SkipClickButton` the player can still advance via keyboard (the top-level widget captures keys directly).

{% hint style="warning" %}
`SkipClickButton` must be named exactly that — otherwise button clicks will be silent.
{% endhint %}

## Configuring the platform hint

In the Blueprint defaults of your SkipButton subclass:

```ini
KeyboardHint = "Press Space"   ; for keyboard/mouse
GamepadHint  = "Press A"       ; for gamepad
```

Call `SetInputDevice(bool bIsGamepad)` when the active input device changes. `SkipLabel` is updated automatically.

```text
Event On Input Device Changed (bIsGamepad)
  → SkipButton → SetInputDevice(bIsGamepad)
```

For the current text: `GetSkipHintText()` returns the active hint.

## Blueprint event

```cpp
// Fires when RequestSkip() is called. For custom animations and sounds.
void OnSkipRequested()
```

```text
Event On Skip Requested
  → Play Sound (UI_Click)
  → Play Animation "ButtonPulse"
```

> 📸 **Image placeholder:** `skip-button-event-graph.png` — Blueprint graph of WBP_MySkipButton. Event "On Skip Requested" → Play Sound + Play Animation. Next to it: Blueprint Defaults panel with KeyboardHint "Press Space" and GamepadHint "Press A" visible.
> *Setup:* Take a screenshot of WBP_MySkipButton → Event Graph + Blueprint Defaults.

## Typical layout

```
WBP_SkipButton
└── Overlay
    ├── Background (darkened corner)
    ├── Icon Image  (platform-dependent: Space / Ⓐ / touch symbol)
    └── SkipLabel   (TextBlock, Name: "SkipLabel")
```

> 📸 **Image placeholder:** `skip-button-umg-designer.png` — UMG Designer of WBP_MySkipButton. Hierarchy: Overlay → Image (Background) + Image (Icon) + TextBlock (Name "SkipLabel"). Viewport preview shows the finished layout.
> *Setup:* Open WBP_MySkipButton in the UMG Designer, take a screenshot of the hierarchy and viewport preview.

## Building your own skip button

**Step 1** — Blueprint subclass: Parent `MayDialogueWidget_SkipButton`, name e.g. `WBP_MySkipButton`.

**Step 2** — UMG Designer: `Button` (Name: `SkipClickButton`), `TextBlock` (Name: `SkipLabel`), plus custom icons and background.

**Step 3** — Set `KeyboardHint` and `GamepadHint` in the Blueprint defaults.

**Step 4** — Optional: implement `On Skip Requested` for sound or animation feedback.

**Step 5** — In your DialogFrame, replace the `SkipWidget` child with `WBP_MySkipButton`.
