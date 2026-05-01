---
description: All fields of UMayDialogueSettings — types, defaults, meaning.
---

# Project Settings (Reference)

Compact reference for `UMayDialogueSettings`.

- **Class**: `UMayDialogueSettings` (`UDeveloperSettings`)
- **Config file**: `DefaultGame.ini`
- **Section**: `/Script/MayDialogue.MayDialogueSettings`
- **UI path**: *Edit → Project Settings → Plugins → MayDialogue*

> 📸 **Image placeholder:** `project-settings-panel.png` — Screenshot of the Project Settings panel.
> *Setup:* Editor open, Edit → Project Settings → Plugins → MayDialogue. Full screenshot of the settings panel with all categories visible: Widget, UMG Component Defaults, Dialogue Defaults, Typewriter, Input, Audio, Babel Voice, Camera. A red arrow points to the navigation path at the top of the settings window.

---

## Widget

| Property | Type | Default | Meaning |
|---|---|---|---|
| `DefaultDialogueWidgetClass` | `TSoftClassPtr<UMayDialogueWidget>` | *(empty)* | UMG widget displayed as the dialogue overlay. Empty → Slate debug fallback. |
| `bUseSlateDialogueWidget` | `bool` | `true` | Show the Slate debug widget while `DefaultDialogueWidgetClass` is empty. |
| `PanelBlurStrength` | `float` | `4.0` | Blur strength of Slate panels (only effective in the Slate debug widget). |

---

## UMG Component Defaults

Fallback classes for the component-based UMG workflow. If `DefaultDialogueWidgetClass` has `BindWidget` slots, those take priority.

| Property | Type |
|---|---|
| `DefaultDialogFrameClass` | `TSoftClassPtr<UMayDialogueWidget_DialogFrame>` |
| `DefaultSpeakerWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Speaker>` |
| `DefaultTextWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Text>` |
| `DefaultChoiceButtonClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceButton>` |
| `DefaultChoiceListClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceList>` |
| `DefaultSkipButtonClass` | `TSoftClassPtr<UMayDialogueWidget_SkipButton>` |

---

## Dialogue Defaults

| Property | Type | Default | Meaning |
|---|---|---|---|
| `DefaultAdvanceMode` | `EMayDialogueAdvanceMode` | `Manual` | Advance mode for SayLines without an explicit override. |
| `DefaultAutoAdvanceDelay` | `float` | `3.0` | Delay in seconds for AdvanceMode `Timer`. |

---

## Typewriter

| Property | Type | Default | Meaning |
|---|---|---|---|
| `bEnableTypewriterEffect` | `bool` | `true` | Global typewriter toggle. |
| `TypewriterCharsPerSecond` | `float` | `30.0` | Characters per second (default, without the `<speed>` inline tag). |

---

## Input

| Property | Type | Default | Meaning |
|---|---|---|---|
| `bAllowSkipTypewriter` | `bool` | `true` | Player input instantly reveals the full text. |
| `bAllowSkipVoiceLine` | `bool` | `false` | Player input stops the running voice and advances. |
| `bSwitchToUIInputDuringDialogue` | `bool` | `true` | Input mode `GameAndUI` for the duration of the dialogue. |
| `bShowMouseCursorDuringDialogue` | `bool` | `true` | Show the mouse cursor during the dialogue. |

---

## Audio

| Property | Type | Default | Meaning |
|---|---|---|---|
| `DefaultSoundClass` | `TSoftObjectPtr<USoundClass>` | *(empty)* | Default sound class for voice playback. |
| `DefaultAttenuation` | `TSoftObjectPtr<USoundAttenuation>` | *(empty)* | Default 3D attenuation for voice audio. |
| `bForce2D` | `bool` | `false` | Force all voice playback as 2D (ignores attenuation). |

---

## Babel Voice

| Property | Type | Default | Meaning |
|---|---|---|---|
| `bEnableBabelVoice` | `bool` | `true` | Enable Babel synthesis when no voice asset is set. |
| `DefaultBabelProfile` | `TSoftObjectPtr<UMayDialogueBabelProfile>` | Plugin default | Fallback profile when the speaker has no profile of its own. |

---

## Camera

| Property | Type | Default | Meaning |
|---|---|---|---|
| `bAutoFocusSpeaker` | `bool` | `false` | Automatic CameraFocus on the current SayLine speaker. |
| `DefaultCameraBlendTime` | `float` | `0.75` | Blend duration in seconds for CameraFocus without an explicit value. |

---

## Programmatic Access

```cpp
const UMayDialogueSettings* S = GetDefault<UMayDialogueSettings>();
float Delay = S->DefaultAutoAdvanceDelay;
bool bTypewriter = S->bEnableTypewriterEffect;
```

Blueprint access:

```text
[Get Class Defaults] (Class: MayDialogueSettings) → Read properties
```

---

## Config Example (DefaultGame.ini)

```ini
[/Script/MayDialogue.MayDialogueSettings]
bEnableTypewriterEffect=True
TypewriterCharsPerSecond=42.0
bEnableBabelVoice=True
DefaultAdvanceMode=Manual
DefaultAutoAdvanceDelay=2.5
DefaultCameraBlendTime=0.5
bAutoFocusSpeaker=False
```

{% hint style="info" %}
`TSoftObjectPtr` and `TSoftClassPtr` references are lazy-loaded on the first dialogue start. This causes a brief one-time load pause — plan for this at your first dialogue start.
{% endhint %}

## See Also

- [Editor Settings](editor-settings.md) — editor-specific settings (node colors, debug highlights).
- [UI → UMG Architecture](../ui/umg-architecture.md) — how the UMG defaults control widget layout.
- [Audio → Babel System](../audio/babel-system.md) — Babel details and profile configuration.
