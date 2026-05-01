# Say Line

The SayLine Node has a Speaker say a line. It is the most frequently used Node — every spoken or written dialogue line is a SayLine.

## When should I use it?

- Whenever an NPC or the player character says something.
- For narrator text (set the Speaker tag to a dedicated Narrator Speaker).
- As a response after a player choice.
- With a voice asset for voiced dialogue, without a voice asset for purely text-based dialogue (with optional Babel fallback).
- With EmotionTags when the UI should display the Speaker's emotional state.

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `SpeakerTag` | `FGameplayTag` | empty | Which Speaker says the line. Must match an entry in the asset's Speakers list. |
| `DialogueText` | `FText` | empty | The displayed text. Supports rich-text tags (see below). |
| `DialogueVoice` | `USoundBase*` | empty | Primary voice asset (culture-independent). Empty = no voice. |
| `VoicePerCulture` | `TMap<FString, USoundBase*>` | empty | Voice assets per culture key (e.g. `"de"`, `"en"`). Overrides `DialogueVoice` for the matching culture. |
| `VolumeMultiplier` | `float` | `1.0` | Node-specific volume multiplier (0.0–4.0). |
| `PitchMultiplier` | `float` | `1.0` | Node-specific pitch multiplier (0.1–4.0). |
| `NodeAudioMode` | `EMayDialogueAudioMode` | `Default` | `Default` = inherit from plugin/Speaker setting; `Force2D` = always 2D; `Spatial3D` = always spatial. |
| `EmotionTags` | `FGameplayTagContainer` | empty | Tags for emotion/intensity/scene that the UI can use (e.g. `Dialogue.Emotion.Angry`). |
| `bUseDefaultAdvanceMode` | `bool` | `true` | When active, the global advance mode from the plugin settings applies. |
| `AdvanceModeOverride` | `EMayDialogueAdvanceMode` | `Manual` | Active only when `bUseDefaultAdvanceMode = false`. Options: `Manual`, `Timer`, `AfterVoice`, `AfterAnimation`, `Immediate`. |

{% hint style="info" %}
**Inline editing:** Double-clicking the SayLine Node in the graph opens a text editor directly on the Node — you do not need to open the Details panel.
{% endhint %}

> 📸 **Image placeholder:** `sayline-node-graph.png` — SayLine Node in the graph with example values.
> *Setup:* A SayLine Node selected. Visible on the Node: Speaker tag `Dialogue.Speaker.Guard`, text `"Halt! Who are you?"`, EmotionTag pill `Dialogue.Emotion.Suspicious`. Input pin connected to the Entry Node on the left, Output pin connected to the PlayerChoice Node on the right.

> 📸 **Image placeholder:** `sayline-details-panel.png` — Details panel of a SayLine.
> *Setup:* Select SayLine. In the Details panel: `SpeakerTag = Dialogue.Speaker.Guard`, `DialogueText = "Halt! Who are you?"`, `DialogueVoice` (empty), `VolumeMultiplier = 1.0`, `PitchMultiplier = 1.0`, `NodeAudioMode = Default`, `EmotionTags = (Dialogue.Emotion.Suspicious)`, `bUseDefaultAdvanceMode = true`.

## Rich-Text Tags

The following inline tags are permitted in `DialogueText`:

| Tag | Effect |
| --- | --- |
| `<pause=0.5>` | 0.5 s pause in the typewriter. |
| `<speed=2.0>` | Doubles the typewriter speed (reset at the end of the line). |
| `<shake>…</shake>` | Text shakes. |
| `<wave>…</wave>` | Text waves. |
| `<color=#FF0000>…</color>` | Color. |
| `<b>…</b>` | Bold. |

Details: [UI → Rich-Text Tags](../../ui/rich-text-tags.md).

## Mini Example

```text
[Entry]
  │
  ▼
[SayLine: Guard | "Halt! <b>Who</b> are you?" | EmotionTag: Suspicious]
  AdvanceModeOverride: Manual
  │
  ▼
[PlayerChoice]
  ├─ Choice 0 "A friend of the king."
  └─ Choice 1 "That is none of your business."
```

> 📸 **Image placeholder:** `sayline-example-graph.png` — Mini-graph Entry → SayLine → PlayerChoice.
> *Setup:* Graph with three Nodes: `Entry` → `SayLine "Halt! Who are you?"` (SpeakerTag Guard, EmotionTag Suspicious) → `PlayerChoice` with two Choice pills. Connections: Entry Output → SayLine Input, SayLine Output → PlayerChoice Input.

## Common Pitfalls

- **SpeakerTag empty or wrong**: The SayLine is displayed without Speaker data (no portrait, no name). Make sure the tag matches a Speaker in the Speakers list.
- **`bUseDefaultAdvanceMode = true` forgotten**: If you want a different advance mode for individual Nodes, first disable `bUseDefaultAdvanceMode`, otherwise `AdvanceModeOverride` has no effect.

## Extending

{% hint style="info" %}
**Custom variant:** Derive a Blueprint subclass from `UMayDialogueNode_SayLine` and override `ExecuteClientEffects` to, for example, start a lipsync controller or address a custom audio engine.
{% endhint %}
