---
description: Play a non-voice sound — 2D stinger, 3D ambient, or SFX accent.
---

# Play Sound

Plays any `USoundBase` (SoundCue, SoundWave, or MetaSound). This Node is exclusively for **non-voice audio** — thunder, door creaks, heartbeat, music stingers. Voice belongs in the voice slot of the SayLine.

## When to use

- **Atmospheric accent** — Thunderclap between two lines, 2D, no spatial setup needed.
- **3D sound effect on NPC** — Chains rattle, sword draws, footsteps — attached to the Participant's position.
- **Music stinger** — Short dramatic motif directly before a reveal line.
- **Horror atmosphere** — Heartbeat loop (fire-and-forget), dialogue continues in parallel.

---

> 📸 **Image placeholder:** `play-sound-node.png` — "Play Sound" Node in the MayDialogue graph.
> *Setup:* Node alone, title bar "Play Sound" (category color: teal/Audio). Subtitle shows: `Sound = SC_Thunder`, `NodeAudioMode = Force2D`. Input and Output pins visible.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `Sound` | `USoundBase*` | The sound asset. Supports SoundCue, SoundWave, and MetaSound. |
| `VolumeMultiplier` | `float` | Volume multiplier. Range: 0–4. Default: `1.0`. |
| `PitchMultiplier` | `float` | Pitch multiplier. Range: 0.1–4. Default: `1.0`. |
| `NodeAudioMode` | `EMayDialogueAudioMode` | `Default` = inherit plugin setting. `Force2D` = always flat. `Spatial3D` = always 3D at Instigator/Target. |
| `AttenuationOverride` | `USoundAttenuation*` | Node-specific attenuation preset. Overrides plugin default. Only active in 3D mode. |

---

> 📸 **Image placeholder:** `play-sound-details.png` — Details panel with 3D setup.
> *Setup:* Select the Node. In the Details panel: `Sound = SC_ChainRattle`, `VolumeMultiplier = 1.2`, `PitchMultiplier = 1.0`, `NodeAudioMode = Spatial3D`, `AttenuationOverride = ATT_NPC_Close`.

---

## Action Node or SideEffect Sub-Node?

If the sound is **the dramatic main point** of the step (the thunderclap is intentionally placed between two SayLines as its own moment), use the Action Node. If the sound is just incidentally assigned to a SayLine (NPC knocks on the table while speaking), attach it as a SideEffect pill to the SayLine.

---

## Example: Atmospheric transition

```text
[SayLine: Player "I have a bad feeling..."]
  │
  ▼
[PlaySound: SC_Thunder, NodeAudioMode=Force2D, VolumeMultiplier=1.5]
  │
  ▼
[SayLine: Player "...and it's only getting worse."]
```

> 📸 **Image placeholder:** `play-sound-example-graph.png` — Graph snippet of the thunder example.
> *Setup:* Three Nodes: SayLine (Player) → PlaySound (SC_Thunder, Force2D in subtitle) → SayLine (Player). All pins connected.

---

## Pitfalls

{% hint style="info" %}
**Voice lines do not belong here.** Voice audio goes into the voice slot of the SayLine — not into PlaySound. PlaySound is for everything except speech.
{% endhint %}

- The Node is always **pass-through** — long sounds continue playing, the dialogue advances immediately. For a "wait until sound is done" pattern there is currently no native mechanism; use a `Wait` Node with an estimated duration or control the length via SoundCue.
- `NodeAudioMode = Default` inherits the project-wide `bForce2D` setting from the Project Settings.
- If `NodeAudioMode = Spatial3D` and no Participant tag is set, the Instigator position is used as the play location.
- `AttenuationOverride` only has effect in effective 3D mode — it is ignored when `Force2D` is active.
