---
description: All settings under Edit → Project Settings → Plugins → MayDialogue explained.
---

# Project Settings

MayDialogue registers two settings sections:

* **MayDialogue**: Runtime configuration. Saved to `DefaultGame.ini` and frozen in shipping builds. Configure everything that affects the running game here.
* **MayDialogue Editor**: Editor configuration. Saved to `EditorPerProjectUserSettings.ini` and only applies in the editor. Configure node colors and editor behavior here.

Open both under **Edit → Project Settings → Plugins**.

> 📸 **Image placeholder:** `settings-overview.png`: Project Settings with both MayDialogue sections in the sidebar.
> *Setup:* Project Settings window. In the left category bar under "Plugins" two entries are visible: "MayDialogue" and "MayDialogue Editor". Red arrow pointing at both entries.

---

## Runtime Settings: MayDialogue

### UI

| Property | Type | Default | When to change |
| --- | --- | --- | --- |
| `DefaultDialogueWidgetClass` | `TSoftClassPtr<UMayDialogueWidget>` | *(none)* | When you want to use your own UMG widget. Leave empty to get the Slate debug widget. |
| `bUseSlateDialogueWidget` | bool | `true` | Set to `false` once you have set `DefaultDialogueWidgetClass` and no longer need the debug widget. |
| `PanelBlurStrength` | float | `10.0` | Blur strength of the Slate debug panel. Only relevant while using the debug widget. |

{% hint style="info" %}
**Quickest choice to get started:** Leave `DefaultDialogueWidgetClass` empty and keep `bUseSlateDialogueWidget = true`. The debug widget appears automatically, no widget Blueprint needed. Swap it for your own widget once the design is ready.
{% endhint %}

### UI: Component Defaults

If you use `DefaultDialogueWidgetClass` but don't want to hardcode all sub-widgets as `BindWidget` inside it, you can set fallback classes for each slot:

| Property | Purpose |
| --- | --- |
| `DefaultDialogFrameClass` | Main container (background, position) |
| `DefaultSpeakerWidgetClass` | Portrait + speaker name |
| `DefaultTextWidgetClass` | Typewriter text field |
| `DefaultChoiceButtonClass` | Individual choice button |
| `DefaultChoiceListClass` | Container for all choice buttons |
| `DefaultSkipButtonClass` | "Next / Skip" button |

Architecture details: [UI → UMG Architecture](../ui/umg-architecture.md).

### Dialogue Defaults

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `DefaultAdvanceMode` | Enum | `Manual` | How SayLines advance by default. `Manual` = player click, `Timer` = after N seconds, `AfterVoice` = when audio ends, `Immediate` = immediately. |
| `DefaultAutoAdvanceDelay` | float | `3.0` | Wait time in seconds when `DefaultAdvanceMode = Timer`. |

{% hint style="info" %}
`DefaultAdvanceMode` is the global fallback. Each SayLine node can override this value via `AdvanceModeOverride`.
{% endhint %}

### Typewriter

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `bEnableTypewriterEffect` | bool | `true` | Global switch. `false` = all text appears instantly. |
| `TypewriterCharsPerSecond` | float | `30.0` | Characters per second. Can be overridden per-text with the `<speed>` rich text tag. |

### Input

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `bAllowSkipTypewriter` | bool | `true` | Player input shows the full text immediately. |
| `bAllowSkipVoiceLine` | bool | `true` | Player input stops the current voice playback and moves to the next node. |
| `bSwitchToUIInputDuringDialogue` | bool | `true` | Sets input mode to `Game+UI` so cursor and UI interaction work. |
| `bShowMouseCursorDuringDialogue` | bool | `true` | Shows the cursor for choice buttons. Only active when `bSwitchToUIInputDuringDialogue = true`. |

{% hint style="info" %}
**Note for advanced setups with custom input modes:** The input mode is hard-coded back to `GameOnly` after the dialogue ends. If your game uses a different mode before the dialogue (`UIOnly`, `GameAndUI`), you need to restore it manually after `OnDialogueEnded`. For most projects the default behavior is correct. See [Known Issues](../troubleshooting/known-issues.md).
{% endhint %}

### Audio

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `DefaultSoundClass` | `TSoftObjectPtr<USoundClass>` | *(none)* | SoundClass for all dialogue voices when no speaker override applies. Point this at your project's "Voice" SoundClass. |
| `DefaultAttenuation` | `TSoftObjectPtr<USoundAttenuation>` | *(none)* | 3D attenuation for voices. Point this at your base attenuation asset. |
| `bForce2D` | bool | `false` | Play all dialogue voices in 2D (visual novel mode). |

### Babel Voice

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `bEnableBabelVoice` | bool | `false` | Enables procedural placeholder voices for SayLines without a voice asset. |
| `DefaultBabelProfile` | `TSoftObjectPtr<UMayDialogueBabelProfile>` | *(plugin default)* | Fallback profile for speakers without their own BabelProfile. |

Details: [Audio → Babel System](../audio/babel-system.md).

### Camera

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `bAutoFocusSpeaker` | bool | `false` | Camera automatically pans to the current speaker without needing a CameraFocus node. |
| `DefaultCameraBlendTime` | float | `0.5` | Blend duration in seconds when a CameraFocus node specifies no time of its own. |

### GAS: Lifecycle Cue Bindings

| Property | Type | Meaning |
| --- | --- | --- |
| `LifecycleCueBindings` | `TArray<FMayDialogueCueLifecycleBinding>` | Declarative table: when dialogue event X occurs, fire GameplayCue Y on the instigator's ASC. |

Each entry (`FMayDialogueCueLifecycleBinding`) has:

| Field | Meaning |
| --- | --- |
| `LifecycleEvent` | When the cue fires: `DialogueStarted`, `DialogueEnded`, `NodeReached`, `ChoiceMade`, `DialogueEventFired` |
| `EventTagFilter` | Only for `DialogueEventFired`: fires only when the event tag matches. Empty = always. |
| `CueTag` | The GameplayCue tag (must be under the `GameplayCue` hierarchy) |
| `Mode` | `Execute` (one-shot), `Add` (persistent), `Remove` |

This lets you fire, for example, a blur cue when a dialogue starts, without writing any Blueprint code.

> 📸 **Image placeholder:** `settings-runtime-panel.png`: Project Settings with the MayDialogue panel filled in.
> *Setup:* Project Settings → MayDialogue. Visible: sections "UI", "Dialogue Defaults", "Typewriter", "Input", "Audio", "Babel Voice", "Camera", "GAS Lifecycle Cues". Key fields filled in: `DefaultSoundClass` and `DefaultAttenuation` pointing to project assets, `bEnableTypewriterEffect = true`, `DefaultAdvanceMode = Manual`.

---

## Editor Settings: MayDialogue Editor

### Node Colors

Here you can customize the default color of each node type in the graph editor. The setting is per-project and saved to `EditorPerProjectUserSettings.ini`.

Available properties: `SayLineColor`, `PlayerChoiceColor`, `BranchColor`, `RandomLineColor`, `WaitColor`, `LinkColor`, `SubGraphColor`, `CameraFocusColor`, `AnimationColor`, `VariableColor`, `LogicColor`, `EntryExitColor`, `ErrorColor`.

{% hint style="info" %}
Speaker colors are defined **per dialogue asset** in the Speakers panel and override `SayLineColor` for the title bar of that speaker's SayLine nodes. `SayLineColor` applies only to SayLines with no speaker configured.
{% endhint %}

### Debug Highlights

| Property | Meaning |
| --- | --- |
| `ActiveDebugColor` | Highlight color for the node currently paused in the PIE debugger. |
| `HistoryDebugColor` | Highlight color for all nodes already visited in this debug session. |

### Editor Behavior

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `bAutoCompileOnSave` | bool | `true` | Saving a dialogue asset automatically runs a compile pass. Errors appear immediately in the Compiler Results panel. |
| `bShowMinimapByDefault` | bool | `false` | Reserved for a future minimap view. |

---

## Recommended minimum configuration for a new project

When starting a project from scratch, these four settings are enough to begin:

1. Point **`DefaultSoundClass`** at your project's voice SoundClass.
2. Point **`DefaultAttenuation`** at your base attenuation asset.
3. Point **`DefaultSpeakerWidgetClass`** and **`DefaultDialogFrameClass`** at your Blueprint subclasses once you have built initial UI widgets.
4. Everything else can stay at defaults until you need to change something specific.

> 📸 **Image placeholder:** `settings-minimum-config.png`: Project Settings with the minimum configuration for a new project.
> *Setup:* MayDialogue settings panel. Only `DefaultSoundClass` and `DefaultAttenuation` are populated with assets (arrows pointing at project assets). All other fields are empty or at default. Red arrows on `DefaultSoundClass` and `DefaultAttenuation`.
