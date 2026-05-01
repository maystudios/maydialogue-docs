---
description: Fine-tuning individual lines or sound events — 2D override for inner monologues, volume and pitch per PlaySound.
---

# Node Overrides

## When Do I Need This?

Node overrides are for **exceptions**. You have already set speaker overrides, but a single line needs to sound different:

- This one SayLine is an inner monologue → 2D instead of 3D
- This ambient sound should be quieter than normal
- This scream should sound higher than the rest

Everything that applies **only to this one line** belongs on the node. Everything that applies to the character in general belongs on the [Speaker Override](speaker-overrides.md).

## Overrides on SayLine

> 📸 **Image placeholder:** `node-overrides-sayline-details.png` — Details panel of a SayLine with NodeAudioMode = Force2D.
> *Setup:* Select a SayLine node in the graph editor. Details panel on the right shows: `NodeAudioMode = Force2D` (dropdown, selected). Above it `SpeakerTag`, `DialogueText`, and further SayLine properties below. Red arrow on `NodeAudioMode`.

| Property | Type | Effect |
|---|---|---|
| `NodeAudioMode` | Enum | `Default` / `Spatial3D` / `Force2D` for this line |

Volume and pitch overrides exist at the speaker level. If you want to make individual lines louder or quieter, create a dedicated speaker entry (e.g. `Dialogue.Speaker.InnerVoice`) and set `VolumeMultiplier` / `PitchMultiplier` there.

## Overrides on PlaySound

The `PlaySound` node can play audio independently — separate from SayLines. It has full override control.

> 📸 **Image placeholder:** `node-overrides-playsound-details.png` — Details panel of a PlaySound node with filled values.
> *Setup:* Select a PlaySound node in the graph. Details panel on the right shows: `Sound = SFX_Heartbeat`, `VolumeMultiplier = 0.4`, `PitchMultiplier = 1.0`, `bForce2D = false`, `TargetTag = Dialogue.Speaker.Guard`. All fields filled, red arrow on `VolumeMultiplier`.

| Property | Type | Effect |
|---|---|---|
| `Sound` | USoundBase | The sound to play (SoundWave, SoundCue, MetaSound) |
| `VolumeMultiplier` | float | Volume scale for this sound |
| `PitchMultiplier` | float | Pitch scale for this sound |
| `bForce2D` | bool | 2D playback (ignores attenuation) |
| `TargetTag` | FGameplayTag | Optional: play the sound at the actor of this speaker |

## Advance Mode and Audio Timing

The `AdvanceModeOverride` on a SayLine controls how the dialogue flow handles audio ending:

| Mode | Audio behaviour |
|---|---|
| `Manual` | Audio plays; the player can advance independently |
| `Timer` | Dialogue advances after `AutoAdvanceDelay`, regardless of audio |
| `AfterVoice` | Dialogue advances **exactly** when the voice asset finishes |
| `AfterAnimation` | Dialogue advances when the montage ends (audio plays in parallel) |
| `Immediate` | No waiting — audio starts but may be immediately replaced by the next node |

`AfterVoice` is the right mode when audio and dialogue flow should stay in sync.

## Practical Examples

### Inner Monologue — 2D Instead of 3D

```text
SayLine "I should run."
  NodeAudioMode = Force2D
```

This line plays directly into the player's ear without spatial positioning — correct for thoughts only the player hears.

> 📸 **Image placeholder:** `node-overrides-inner-monologue-graph.png` — Graph section with SayLine "I should run." and NodeAudioMode = Force2D in the details panel.
> *Setup:* Graph editor with SayLine node selected. Node title shows speaker and text. Details panel on the right: `NodeAudioMode = Force2D` expanded. Further nodes of the dialogue visible in the background. Arrow visible: previous node → `SayLine "I should run."` → next node.

### Ambient Sound Quieter

```text
PlaySound
  Sound            = SFX_AmbientDrip
  VolumeMultiplier = 0.4
  bForce2D         = false
```

The dripping sound should stay in the background, not dominate.

### Panicked Scream — Higher Pitch

```text
PlaySound
  Sound            = SFX_Scream
  PitchMultiplier  = 1.3
  VolumeMultiplier = 1.1
```

A higher frequency underlines the urgency of the moment.

## Node Override vs. Speaker Override

```text
Rule of thumb: If it applies only to this one line → Node Override.
               If it applies to the character in general → Speaker Override.
```

| Situation | Correct level |
|---|---|
| All lines of the ghost sound reverberant | Speaker Override |
| Only the ghost's final line is 2D | Node Override (NodeAudioMode) |
| Background sound should be quieter | PlaySound with VolumeMultiplier |
