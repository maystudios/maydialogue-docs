---
description: Set a custom SoundClass, attenuation, 2D/3D mode, volume, and pitch per speaker — configure once, consistent everywhere.
---

# Speaker Overrides

## When Do I Need This?

When a character should **always** sound different from the rest — regardless of which line they say. Typical cases:

- The ghost whispers quietly and reverberates
- The narrator never speaks spatially, always directly into the ear
- The monster has a lower base pitch
- The protagonist is generally a little quieter than the NPCs

Speaker overrides set this sonic identity **once** in the Speakers panel. From then on it applies automatically to all SayLines of that speaker — without touching each node individually.

## Where to Configure

Speakers panel in the dialogue asset → expand the speaker entry → "Audio Overrides" section.

> 📸 **Image placeholder:** `speaker-overrides-panel.png` — Speakers panel with an expanded speaker entry and the audio override fields visible.
> *Setup:* Open the dialogue asset, Speakers panel on the left. Expand the "Ghost" speaker entry. In the expanded area: `AudioModeOverride` (dropdown), `SoundClassOverride` (asset slot), `AttenuationOverride` (asset slot), `VolumeMultiplier` (float), `PitchMultiplier` (float), `BabelProfile` (asset slot). All fields filled with example values (SC_VoiceGhost, Att_Whisper, 0.7, 0.85).

## Available Properties

| Property | Type | What it does |
|---|---|---|
| `AudioModeOverride` | Enum | `Default` (from plugin), `Spatial3D` (force 3D), `Force2D` (always 2D) |
| `SoundClassOverride` | USoundClass | Mixer class for this speaker (e.g. dedicated reverb bus) |
| `AttenuationOverride` | USoundAttenuation | 3D radius, occlusion sensitivity for this speaker |
| `VolumeMultiplier` | float | Multiplier on the plugin default volume |
| `PitchMultiplier` | float | Multiplier on the plugin default pitch |
| `BabelProfile` | UMayDialogueBabelProfile | Babel profile when no voice asset is present |

## Practical Examples

### The Ghost Sounds Ghostly

```text
SoundClassOverride  = SC_VoiceGhost    ← dedicated reverb send bus
AttenuationOverride = Att_GhostWhisper ← short radius, high occlusion sensitivity
PitchMultiplier     = 0.85             ← slightly lower than normal
VolumeMultiplier    = 0.7              ← noticeably quieter
```

All SayLines of the ghost sound like this — no node-by-node setup required.

> 📸 **Image placeholder:** `speaker-overrides-ghost-filled.png` — Details panel of the "Ghost" speaker with filled values.
> *Setup:* Open the dialogue asset, expand the Ghost speaker. Visible in the details area: `AudioModeOverride = Default`, `SoundClassOverride = SC_VoiceGhost`, `AttenuationOverride = Att_GhostWhisper`, `VolumeMultiplier = 0.7`, `PitchMultiplier = 0.85`. Fields clearly annotated with red arrows pointing to the set values.

### The Narrator Always Speaks in 2D

```text
AudioModeOverride   = Force2D
SoundClassOverride  = SC_VoiceNarrator  ← has dialogue ducking
```

Narrator lines are never positioned spatially — correct for off-screen narrators.

### The Protagonist Is Quieter

```text
VolumeMultiplier = 0.9
```

All NPCs run at `1.0`, the player character is minimally pulled back — a subtle mix.

### The Monster Has a Procedural Voice

```text
BabelProfile = BP_Babel_Monster  ← PhonemeBase mode, low frequency
```

No voice asset present; Babel takes over — every line sounds like creature growling.

## Consistency Across Multiple Dialogue Assets

Speaker overrides live **in the dialogue asset**, not globally. This means: if you use the same character in ten different assets, the overrides need to be identical in each one.

{% hint style="info" %}
**Tip:** Maintain a DataTable with speaker templates (SoundClass, Attenuation, Pitch, Volume, BabelProfile per character). Copy-paste the row into new dialogue assets. This keeps sonic identity consistent without retyping it by hand.

How to create the DataTable: Content Browser → right-click → Miscellaneous → Data Table → choose row struct `FMayDialogueSpeaker`. Each row represents a character audio profile.
{% endhint %}

> 📸 **Image placeholder:** `speaker-overrides-datatable.png` — DataTable with speaker audio templates in the Content Browser.
> *Setup:* Content Browser, open a DataTable asset (row struct with the audio override fields). Visible: at least three rows (Ghost, Narrator, Monster) with SoundClass, Attenuation, Volume, and Pitch values filled per row. Table layout clearly readable.

## Speaker Override vs. Node Override

| Situation | Correct level |
|---|---|
| Character **always** sounds like this | Speaker Override |
| Only **this one line** is different | [Node Override](node-overrides.md) |
| Applies to an entire scene, multiple speakers | Adjust the plugin default or create an alternative speaker entry |

{% hint style="warning" %}
`AttenuationOverride` on a speaker always sets the attenuation — even if the actor itself had a `UAudioComponent` with its own attenuation setting. The speaker override takes precedence.
{% endhint %}
