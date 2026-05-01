---
description: Procedural nonsense voices when no voice asset is available — as a development placeholder or a permanent sonic style.
---

# Babel System

## What Is Babel?

Babel automatically generates acoustic accompaniment for every dialogue line that has no voice asset. The result sounds like speech but is not a real word — "nonsense speech".

**Two typical use cases:**

- **Placeholder during development:** Babel bridges the gap until real voice recordings are available. No silence in PIE testing, no missing audio flags — the dialogue already sounds like something.
- **Permanent style:** Babel as a deliberate design choice. Animal Crossing blips, Fears-to-Fathom murmuring, unsettling creature phonemes — Babel is designed to remain in the finished game if you want.

## When Does Babel Kick In?

Babel is enabled by default. It activates for a SayLine when:

1. The `DialogueVoice` map has no entry for the current culture **and**
2. No default voice asset is present

Global switch in the Project Settings:

> 📸 **Image placeholder:** `babel-project-settings.png` — Project Settings with the MayDialogue audio section, focus on bEnableBabelVoice and DefaultBabelProfile.
> *Setup:* Editor → Edit → Project Settings → Plugins → MayDialogue → Audio. Visible: `bEnableBabelVoice = true` (checkbox ticked), `DefaultBabelProfile` as an asset slot with `BP_Babel_Default` assigned. Red arrow pointing to both fields.

| Setting | Effect |
|---|---|
| `bEnableBabelVoice` | Babel on/off globally (default: `true`) |
| `DefaultBabelProfile` | Profile when no speaker profile is set |
| `BabelEngine` | Synthesis engine: `Granular` (recommended, sample-pool based) or `BiquadLegacy` (original sinusoidal DSP path). Configurable via `EMayDialogueBabelEngine`. |

{% hint style="info" %}
`BabelEngine = Granular` uses the pre-recorded sample pool in `Content/DefaultBlips/Sounds/` for higher audio quality (Fears-to-Fathom / Animal Crossing style). `BiquadLegacy` preserves the original procedural DSP path.
{% endhint %}

## Two Synthesis Modes

Babel can sound in two different ways:

| Mode | What it sounds like | Typical use |
|---|---|---|
| **BlipPerCharacter** | A short sound per revealed character ("bip bip bip") | VN blips, Animal Crossing style, friendly NPCs |
| **PhonemeBase** | Sinusoidal tones based on vowel/consonant type | Abstract creatures, unsettling voices, non-human entities |

## Two Sync Modes

How Babel synchronises with the typewriter text:

| Mode | How | When |
|---|---|---|
| **TypewriterSync** | Reacts to every letter revealed by the typewriter | Standard — precise synchronisation, each blip with the correct character |
| **Continuous** | Internal timer, independent of the typewriter | When you are not using a typewriter or are simulating creature growling |

The typical case is **BlipPerCharacter + TypewriterSync**: one blip per character, in sync with the text animation.

## Babel per Speaker

Every speaker can have its **own Babel profile**. This is the core of Babel's flexibility:

- The guard has short, deep blips
- The child has high, fast blips
- The ghost uses PhonemeBase at a low frequency
- The monster NPC has no blips but a continuous phoneme growl pattern

Setting a Babel profile on a speaker: Speakers panel → expand the speaker → `BabelProfile` → select an asset.

> 📸 **Image placeholder:** `babel-speaker-profile-assignment.png` — Speakers panel with two different speakers, each with their own BabelProfile asset.
> *Setup:* Open a dialogue asset, Speakers panel. Two speakers expanded and visible: "Guard" with `BabelProfile = BP_Babel_Guard`, "Ghost" with `BabelProfile = BP_Babel_Ghost`. Asset icons clearly identifiable, different asset names show individuality.

## Creating Babel Profiles

A Babel profile is a DataAsset. You can create it as a plain DataAsset **or** as a Blueprint subclass — `UMayDialogueBabelProfile` is now `Blueprintable`, meaning you can build procedural profiles that change their parameters at runtime (e.g. affection level → pitch):

**As a DataAsset (standard):**
1. Content Browser → right-click → **DataAsset**
2. Parent class: `UMayDialogueBabelProfile`
3. Name it (e.g. `BP_Babel_Ghost`)
4. Open and set the parameters

**As a Blueprint subclass (procedural):**
1. Content Browser → right-click → **Blueprint Class**
2. Parent class: `UMayDialogueBabelProfile`
3. Set properties in `Construction Script` or `Event BeginPlay`

> 📸 **Image placeholder:** `babel-profile-asset-open.png` — Open BabelProfile asset in the details panel with filled values.
> *Setup:* Content Browser → double-click DataAsset `BP_Babel_Ghost`. Details panel shows all properties: `BabelMode = PhonemeBase`, `SyncMode = Continuous`, `PhonemeBaseFrequency = 120`, `PhonemeFrequencyRange = 40`, `PhonemeDuration = 0.12`, `bApplyProsody = true`, `Volume = 0.5`. Red arrow on `BabelMode`.

Details on all properties: [Babel Profiles](babel-profiles.md).

## Preview Runner

In the Preview Runner, Babel is played **2D** automatically — correct for editor testing without any level setup. In PIE with a speaker actor, Babel plays 3D (unless a `Force2D` override is active).

## BabelSynth — Blueprint-Callable Methods

`UMayDialogueBabelSynth` now exposes the following methods directly in Blueprint (category `MayDialogue|Audio`):

| Method | Kind | Description |
| --- | --- | --- |
| `Is Active` | Pure | Checks whether the synth is currently playing |
| `Stop Speech` | Callable | Cancels the currently running Babel output |
| `Set External Volume Multiplier` | Callable | Scales the volume from outside (e.g. for audio ducking) |
| `Set External Pitch Multiplier` | Callable | Scales the pitch from outside (e.g. for stress effects) |

These methods are useful when you control Babel output from another system — for example, muting Babel when a cutscene starts, or lowering pitch for a stunned speaker.

---

## Widget Integration

**SMayDialogueWidget (Slate debug widget, default):** The built-in Slate widget wires `BabelSynth::OnCharacterRevealed` to the typewriter automatically — no manual setup required. This widget is the default fallback when no UMG widget is configured. More details: [Slate Debug Widget](slate-debug-widget.md).

**UMG component path:** In Blueprint you must forward the typewriter events from your text widget to the synth yourself. See [UI Architecture](../ui/umg-architecture.md).

{% hint style="info" %}
**Quality expectation:** Babel is not just a debug placeholder. With custom BlipSounds, PitchVariation, and PunctuationPause, the result can sound good enough that you will want to keep it in the finished game.
{% endhint %}

{% hint style="warning" %}
**Continuous mode in the UMG path:** The synth ticks automatically via Unreal Engine's tick system (`FTickableGameObject`) — you do not need to call anything yourself. In TypewriterSync mode, however, you must forward every revealed character to `OnCharacterRevealed` via C++. In Blueprint you can control the synth using the available nodes:

```text
[Event On Character Revealed]  ← typewriter event from your text widget
    │ Char (int32), CharIndex, TotalChars
    ▼
[Is Active]  ← BabelSynth | MayDialogue|Babel
    │ True
    ▼
[Stop Speech]  ← BabelSynth if needed (e.g. skip button)
```

For simple projects, the SayLine fallback path (without a widget) is the recommended option — Babel starts and stops fully automatically there.
{% endhint %}
