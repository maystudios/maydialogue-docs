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
| `PanelBlurStrength` | `float` | `10.0` | Blur strength of Slate panels (only effective in the Slate debug widget). |

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
| `bAllowSkipVoiceLine` | `bool` | `true` | Player input stops the running voice and advances. |
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
| `DefaultCameraBlendTime` | `float` | `0.5` | Blend duration in seconds for CameraFocus without an explicit value. |

---

## Accessibility (1.0)

| Property | Type | Default | Meaning |
|---|---|---|---|
| `DialogueTextScale` | `float` | `1.0` | Uniform text scale multiplier applied to all dialogue font sizes (message text, speaker name, choice buttons) on both the Slate and UMG widget layers. Clamped to `0.5–3.0`. |
| `bColorblindSafeChoiceCues` | `bool` | `true` | When true, unavailable (locked) choices show a lock glyph prefix so they are distinguishable from available choices without relying on color. |

`DialogueTextScale` is read by `MayDialogue::Accessibility::ClampTextScale(S->DialogueTextScale)` in both the built-in Slate widget (`SMayDialogueWidget`) and the default UMG component widgets. You do not need to replace any widget to use larger text — set the scale here and it applies project-wide.

`bColorblindSafeChoiceCues` prepends a Unicode lock glyph (🔒) to the text of unavailable choice buttons via `MayDialogue::Accessibility::GetColorblindChoicePrefix(...)`. Projects with their own non-color availability indicator may disable this.

**Screen-reader support:** Both widget layers call `SetAccessibleText` on each displayed element using `MayDialogue::Accessibility::MakeSpeakerAccessibleText`, `MakeDialogueTextAccessibleText`, and `MakeChoiceAccessibleText`. The accessible text for unavailable choices includes the phrase "Locked choice" and, when a reason is provided, the requirement description — so assistive technology announces context even without the visual lock glyph.

```ini
# DefaultGame.ini example
[/Script/MayDialogue.MayDialogueSettings]
DialogueTextScale=1.5
bColorblindSafeChoiceCues=True
```

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
DialogueTextScale=1.0
bColorblindSafeChoiceCues=True
```

{% hint style="info" %}
`TSoftObjectPtr` and `TSoftClassPtr` references are lazy-loaded on the first dialogue start. This causes a brief one-time load pause — plan for this at your first dialogue start.
{% endhint %}

---

## Native GameplayTags Shipped by the Plugin (1.0)

The plugin registers a minimal set of native `GameplayTag` values at module startup so the editor tag pickers are populated and the quick-start guide works without any manual tag-table editing.

### Speaker Tags

| C++ Name | Tag String | Purpose |
|---|---|---|
| `TAG_MayDialogue_Speaker_Guard` | `Dialogue.Speaker.Guard` | Example speaker tag — matches the shipped `DA_Greeting_Simple` sample. |
| `TAG_MayDialogue_Speaker_Player` | `Dialogue.Speaker.Player` | Example tag for the player character. |
| `TAG_MayDialogue_Speaker_NPC` | `Dialogue.Speaker.NPC` | Generic example tag for unnamed NPCs. |

### Lifecycle GameplayCue Tags

| C++ Name | Tag String | Maps to |
|---|---|---|
| `TAG_GameplayCue_Dialogue` | `GameplayCue.Dialogue` | Root tag for all MayDialogue lifecycle cues. |
| `TAG_GameplayCue_Dialogue_Started` | `GameplayCue.Dialogue.Started` | `EMayDialogueLifecycleEvent::DialogueStarted` |
| `TAG_GameplayCue_Dialogue_Ended` | `GameplayCue.Dialogue.Ended` | `EMayDialogueLifecycleEvent::DialogueEnded` |
| `TAG_GameplayCue_Dialogue_NodeReached` | `GameplayCue.Dialogue.NodeReached` | `EMayDialogueLifecycleEvent::NodeReached` |
| `TAG_GameplayCue_Dialogue_ChoiceMade` | `GameplayCue.Dialogue.ChoiceMade` | `EMayDialogueLifecycleEvent::ChoiceMade` |
| `TAG_GameplayCue_Dialogue_EventFired` | `GameplayCue.Dialogue.EventFired` | `EMayDialogueLifecycleEvent::DialogueEventFired` |

Tags are declared in `MayDialogueGameplayTags.h` (include to reference by the extern variable names). Definitions live in `MayDialogueGameplayTags.cpp` and are registered automatically.

**Adding project-specific tags:** add your own tags to your project's `DefaultGameplayTags.ini` or via the *Project Settings → GameplayTags* editor alongside the plugin tags. The plugin's tags are intentionally minimal and clearly scoped under `Dialogue.*` so they do not collide with project tag trees.

**Lifecycle cue bindings:** connect the `GameplayCue.Dialogue.*` tags to your GAS-powered cues in *Project Settings → MayDialogue → GAS → Lifecycle Cue Bindings*. See [GAS Integration](../gas/README.md) for the full binding setup.

---

## See Also

- [Editor Settings](editor-settings.md) — editor-specific settings (node colors, debug highlights).
- [UI → UMG Architecture](../ui/umg-architecture.md) — how the UMG defaults control widget layout.
- [Audio → Babel System](../audio/babel-system.md) — Babel details and profile configuration.
- [GAS Integration](../gas/README.md) — lifecycle cue bindings and GAS requirements.
