---
description: How the plugin default, speaker override, and node override interact — and which level wins when.
---

# Three-Level Fallback

Audio settings are applied at three levels. The more specific level always wins.

> 📸 **Image placeholder:** `audio-three-level-diagram.png` — Arrow diagram of the three levels.
> *Setup:* Graphic (not an editor screenshot). Left to right, three boxes with arrows: `[Plugin Default]` → `[Speaker Override]` → `[Node Override]` → `[Effective Audio Config]`. Below each box an italic label: "Project Settings", "Speakers panel in the dialogue asset", "SayLine / PlaySound node". Arrows in dark blue, boxes light grey with white text.

## The Three Levels at a Glance

| Level | Where to configure | Priority |
|---|---|---|
| **Plugin Default** | Project Settings → MayDialogue | Lowest |
| **Speaker Override** | Speakers panel in the dialogue asset | Medium |
| **Node Override** | SayLine or PlaySound node | Highest |

## Level 1 — Plugin Default

Set once per project. Applies to everything that has no override.

Relevant properties in the Project Settings:

| Property | Meaning |
|---|---|
| `DefaultSoundClass` | Mixer class for all dialogue voices |
| `DefaultAttenuation` | 3D spatialization for all speakers |
| `bForce2D` | Force 2D project-wide (e.g. for visual novel mode) |
| `bEnableBabelVoice` | Babel on/off globally |
| `DefaultBabelProfile` | Fallback profile when no speaker profile is set |

> 📸 **Image placeholder:** `audio-project-settings-panel.png` — Project Settings with the MayDialogue audio section.
> *Setup:* Editor → Edit → Project Settings → Plugins → MayDialogue. Visible: Audio category expanded, all properties listed above with their default values. `bEnableBabelVoice` set to `true`, `DefaultSoundClass` and `DefaultAttenuation` as asset references.

## Level 2 — Speaker Override

Applies to **all SayLines of this speaker** in the entire dialogue asset, as long as the node sets no override of its own.

Properties in the Speakers panel:

| Property | Type | Effect |
|---|---|---|
| `AudioModeOverride` | Enum | `Default` / `Spatial3D` / `Force2D` |
| `SoundClassOverride` | SoundClass asset | Mixer class of the speaker |
| `AttenuationOverride` | Attenuation asset | 3D spatialization of the speaker |
| `VolumeMultiplier` | float | Base volume scale of the speaker |
| `PitchMultiplier` | float | Pitch scale of the speaker |
| `BabelProfile` | BabelProfile asset | Profile for Babel synthesis |

## Level 3 — Node Override

Finest control. Applies only to this one SayLine or PlaySound node.

Available on **SayLine**:

| Property | Type | Effect |
|---|---|---|
| `NodeAudioMode` | Enum | `Default` / `Spatial3D` / `Force2D` |

Available on **PlaySound**:

| Property | Type | Effect |
|---|---|---|
| `Sound` | SoundBase asset | Explicit sound for this node |
| `VolumeMultiplier` | float | Volume scale for this sound |
| `PitchMultiplier` | float | Pitch scale for this sound |
| `bForce2D` | bool | Force 2D playback |

## Resolution Algorithm

For every SayLine the plugin runs this sequence:

1. Start with plugin defaults
2. Does the speaker have an override? → overwrite the relevant fields
3. Does the node have an override? → overwrite once more
4. Hand the resulting config to the `UAudioComponent`

## Volume and Pitch Multiply

Volume and pitch multipliers **multiply** across levels — they do not replace each other:

```text
EffectiveVolume = Plugin_Vol × Speaker_Vol × Node_Vol
EffectivePitch  = Plugin_Pitch × Speaker_Pitch × Node_Pitch
```

If the plugin default is `1.0`, the speaker is `0.8`, and a PlaySound node is `0.5`, the result is `0.4`.

## Example: Ghost with Inner Monologue

```text
Plugin Default:
  SoundClass  = SC_Voice
  Attenuation = Att_Default
  bForce2D    = false

Speaker "Ghost":
  SoundClassOverride  = SC_VoiceGhost
  AttenuationOverride = Att_Whisper
  PitchMultiplier     = 0.85
  VolumeMultiplier    = 0.7

Node SayLine "I have been dead for a long time.":
  NodeAudioMode = Force2D   ← inner monologue, directly into the ear
```

**Result for this line:**
- SoundClass = `SC_VoiceGhost` (from speaker)
- Attenuation = ignored (Force2D active)
- Pitch = `0.85` (from speaker)
- Volume = `0.7` (from speaker)
- Playback = 2D (from node)

> 📸 **Image placeholder:** `audio-fallback-ghost-example.png` — Speakers panel with the "Ghost" entry and audio overrides filled in, alongside the SayLine in the graph with NodeAudioMode = Force2D.
> *Setup:* Open a dialogue asset with speaker "Ghost". Speakers panel on the left shows the Ghost entry expanded: `SoundClassOverride`, `AttenuationOverride`, `PitchMultiplier = 0.85`, `VolumeMultiplier = 0.7`. In the graph on the right the SayLine "I have been dead for a long time." is selected, details panel shows `NodeAudioMode = Force2D`.

## When to Use Which Level

| Situation | Correct level |
|---|---|
| All characters should use the same mixer | Plugin Default |
| A character always sounds muffled / reverberant | Speaker Override |
| Exactly this one line is an inner monologue | Node Override |
| Cutscene with a different attenuation radius | Speaker Override with a temporary speaker entry |

{% hint style="info" %}
**Tip for inner monologues:** Create a dedicated speaker entry `Dialogue.Speaker.InnerVoice` and set `AudioModeOverride = Force2D` there. All inner monologues get this speaker — cleanly separated from the 3D character.
{% endhint %}
