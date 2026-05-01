---
description: How MayDialogue plays audio — from the automatic 3D source to the procedural Babel placeholder.
---

# Audio System

When a character stands in the level, their voice comes directly from the level — positioned at the actor, with attenuation, occlusion, and reverb. You do not set up a single AudioComponent by hand.

> 📸 **Image placeholder:** `audio-overview-diagram.png` — Arrow diagram of the audio hierarchy.
> *Setup:* Create a graphic (not an editor screenshot). Top to bottom: `[Plugin Default (Project Settings)]` → arrow right → `[Speaker Override (Dialogue Asset)]` → arrow right → `[Node Override (SayLine / PlaySound)]` → arrow right → `[UAudioComponent on Actor]`. Below the last box two branches: `[3D Spatial (default)]` and `[2D PlaySound (Force2D)]`. Horizontal layout on white background, dark-grey arrows.

## How Does Audio Work in MayDialogue?

### 1. AudioComponent Is Created Automatically

On the first SayLine of a speaker, the plugin creates a `UAudioComponent`, attaches it to the actor, and keeps it for the duration of the dialogue. It is cleaned up when the dialogue ends.

**You do not need to prepare anything.** No component slot on the character, no `BeginPlay` setup.

### 2. 3D First

Default mode: the sound plays at the actor's transform. If the player turns away, the character sounds quieter. If a wall is in between, occlusion kicks in.

### 3. Three-Level Fallback

Every audio property can be set at three levels — from coarse to fine:

| Level | Where to set it | Applies to |
|---|---|---|
| Plugin Default | Project Settings | The entire project |
| Speaker Override | Speakers panel in the dialogue asset | All SayLines of that speaker |
| Node Override | SayLine / PlaySound in the graph | Exactly this one line |

The more specific level wins. Details: [Three-Level Fallback](three-level-fallback.md).

### 4. Babel as a Placeholder or Permanent Style

If no voice asset is set for a line, Babel kicks in: a procedural nonsense-voice engine. It produces either blips per character (Animal Crossing style) or sinusoidal phonemes (unsettling creature voice). Babel is configurable enough that you can keep it in your finished game if you want.

> 📸 **Image placeholder:** `audio-babel-preview-runner.png` — Preview Runner while a dialogue with a Babel speaker is playing.
> *Setup:* Open a dialogue asset with a speaker that has no voice asset. Start the Preview Runner (tab at the bottom of the editor). The SayLine of the Babel speaker is active, typewriter text is running. Visible: text animation, no voice asset in the node details panel, Babel indicator active.

## What the Plugin Covers

- On-demand `UAudioComponent` on the speaker (no manual setup)
- Voice playback as `USoundBase` (SoundWave, SoundCue, MetaSound)
- Culture-based voice resolution (one map per SayLine)
- Babel synthesis (BlipPerCharacter and PhonemeBase)
- Volume, pitch, SoundClass, and attenuation overrides at three levels

## What the Plugin Does Not Do

- No voice recording, no TTS, no DAW import
- No audio bus management beyond the dialogue moment
- No lipsync generation (MetaHuman lipsync is outside the plugin scope)

> 📸 **Image placeholder:** `audio-sayline-preview-button.png` — SayLine node in the dialogue editor with the active Preview button.
> *Setup:* Select a SayLine node, details panel on the right. Visible: `DialogueText`, voice asset slot with a `USoundBase` asset assigned, the **Preview** button (play icon) below it. In the background the graph editor with the open dialogue asset.

## Chapter Overview

- [Three-Level Fallback](three-level-fallback.md) — Plugin / Speaker / Node working together
- [Speaker Overrides](speaker-overrides.md) — individual sonic identity per speaker
- [Node Overrides](node-overrides.md) — exceptions per SayLine or PlaySound
- [Localization (VoicePerCulture)](localization.md) — a different voice asset per culture
- [Babel System](babel-system.md) — procedural nonsense voices
- [Babel Profiles](babel-profiles.md) — configure synthesis parameters per speaker
