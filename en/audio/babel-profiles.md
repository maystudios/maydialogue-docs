---
description: A DataAsset containing all synthesis parameters for a speaker — so every character sounds different, even without voice recordings.
---

# Babel Profiles

A `UMayDialogueBabelProfile` bundles all synthesis parameters for a speaker: mode, pitch, samples, timing, volume. Every speaker can have its own profile.

## Creating a Profile

1. Content Browser → right-click → **May Dialogue → Babel Voice Profile**
2. Name it, e.g. `BP_Babel_Ghost`, `BP_Babel_Guard`, `BP_Babel_Child`
3. Double-click and set the parameters

## All Properties at a Glance

### Mode

| Property | Type | Default | Effect |
|---|---|---|---|
| `BabelMode` | Enum | `BlipPerCharacter` | Synthesis method: blip or phoneme |
| `SyncMode` | Enum | `TypewriterSync` | Timing: typewriter events or continuous |

### Blip Mode Parameters

Active when `BabelMode = BlipPerCharacter`:

| Property | Type | Default | Effect |
|---|---|---|---|
| `BlipSounds` | Array\<USoundBase\> | empty | Pool of sample assets; char index mod array size selects the sample |
| `BasePitch` | float | 1.0 | Base pitch multiplier for all blips |
| `PitchVariation` | float | 0.15 | ± random per blip (0.0 = monotone, 1.0 = very variable) |
| `bVaryPitchByVowel` | bool | true | Vowels receive `VowelPitchMultiplier` |
| `VowelPitchMultiplier` | float | 1.2 | Pitch scale specifically for vowel characters |
| `bSkipSpaces` | bool | true | No blips on spaces |
| `bSkipRepeatedChars` | bool | false | No blip if the character is the same as the previous one |

> 📸 **Image placeholder:** `babel-profiles-blip-params.png` — Open BabelProfile asset (BlipPerCharacter mode) with blip parameters filled in.
> *Setup:* Open DataAsset `BP_Babel_Child`. Details panel: `BabelMode = BlipPerCharacter`, `SyncMode = TypewriterSync`. Below, the blip section: `BlipSounds` filled with 3 assets (BP_Sample_01, 02, 03), `BasePitch = 1.2`, `PitchVariation = 0.35`, `bVaryPitchByVowel = true`, `VowelPitchMultiplier = 1.4`, `bSkipSpaces = true`. All fields clearly readable.

### Phoneme Mode Parameters

Active when `BabelMode = PhonemeBase`:

| Property | Type | Default | Effect |
|---|---|---|---|
| `PhonemeBaseFrequency` | float | 200 Hz | Base frequency of the generated tones |
| `PhonemeFrequencyRange` | float | 100 Hz | Variation per phoneme type |
| `PhonemeDuration` | float | 0.06 s | How long each generated tone sounds |
| `bApplyProsody` | bool | true | Vowels/consonants/sibilants/nasals receive different frequencies |

### Timing

| Property | Type | Default | Effect |
|---|---|---|---|
| `PunctuationPause` | float | 0.15 s | Extra pause after `.` `!` `?` |
| `CommaPause` | float | 0.08 s | Extra pause after `,` |

### Volume

| Property | Type | Default | Effect |
|---|---|---|---|
| `Volume` | float | 0.7 | Master volume of the profile (Babel should not be louder than real voices) |
| `bUseProceduralDefaults` | bool | true | If `BlipSounds` is empty: generate simple sine beeps on the fly |

## Ready-Made Example Profiles

### Friendly NPC — Animal Crossing Style

```text
BabelMode            = BlipPerCharacter
SyncMode             = TypewriterSync
BlipSounds           = [BP_Sample_Boop_01, BP_Sample_Boop_02, BP_Sample_Boop_03]
BasePitch            = 1.1
PitchVariation       = 0.30
bVaryPitchByVowel    = true
VowelPitchMultiplier = 1.3
bSkipSpaces          = true
PunctuationPause     = 0.25
Volume               = 0.6
```

Sounds lively, friendly, and varied. Three soft "boop" samples are enough for variation.

### Unsettling Ghost — Phoneme Growl

```text
BabelMode             = PhonemeBase
SyncMode              = Continuous
PhonemeBaseFrequency  = 120
PhonemeFrequencyRange = 40
PhonemeDuration       = 0.12
bApplyProsody         = true
Volume                = 0.5
```

No typewriter sync, continuous murmuring. Low frequency, little variation = threatening.

### Debug Fallback — Neutral

```text
BabelMode              = BlipPerCharacter
BlipSounds             = (empty)
bUseProceduralDefaults = true
Volume                 = 0.5
```

Generates simple sine beeps. No asset needed, ready to use immediately.

> 📸 **Image placeholder:** `babel-profiles-three-examples.png` — Content Browser with three Babel profile assets side by side: BP_Babel_NPC, BP_Babel_Ghost, BP_Babel_Debug.
> *Setup:* Content Browser filtered to `BP_Babel_`. Three DataAsset icons visible: `BP_Babel_NPC`, `BP_Babel_Ghost`, `BP_Babel_Debug`. Tooltips or asset details show different BabelMode values.

## Assigning a Profile to a Speaker

In the Speakers panel of the dialogue asset:

1. Expand the speaker entry
2. Field `BabelProfile` → asset slot → select the profile

All SayLines of that speaker will now use the profile — as long as no voice asset is present for the current culture.

## Global Fallback

In the Project Settings: `DefaultBabelProfile`. Used when:
- No voice asset is present
- No speaker profile is set
- `bEnableBabelVoice = true`

Set the neutral debug profile here as a base.

## Designer Tips

{% hint style="success" %}
**Record samples at slightly different pitches** (3–5 variants are enough). Babel distributes them by char index — this creates organic variation without pitch-shifting artifacts.
{% endhint %}

{% hint style="info" %}
**Tune BasePitch and PitchVariation together:**
- High variation (0.3–0.5) = lively, playful
- Low variation (0.0–0.1) = monotone, unsettling, robotic
{% endhint %}

{% hint style="warning" %}
**Keep samples short** (under 100 ms). Longer samples overlap during typewriter playback and sound muddy.

**Set Volume consciously below 1.0** — Babel should never be louder than real voices when those are eventually added.
{% endhint %}
