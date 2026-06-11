---
description: Voice-line lifecycle events and MetaSound parameter binding — the hooks a VO/audio professional needs to drive lip-sync, mixing, and procedural voices.
---

# VO Pro Hooks: Voice-Line Events & MetaSound

## When Do I Need This?

When you are wiring dialogue voice into a larger audio or animation system and need precise, frame-accurate hooks:

- A **lip-sync** or facial-animation system needs to know exactly when a voice line starts and stops.
- An **audio mixer** wants to duck music or trigger a "character speaking" snapshot for the duration of each line.
- A **MetaSound** voice asset should react to the line's speaker, emotion, and pitch — e.g. a creature voice that shifts timbre by emotion.

MayDialogue exposes two things for this: **voice-line lifecycle delegates** on the dialogue instance, and **automatic MetaSound parameter binding** when the voice asset is a MetaSound Source.

## Voice-Line Lifecycle Events

Two events fire around every voice line, alongside the existing message events:

| Event | Fires when |
|---|---|
| `OnVoiceLineStarted` | A line's voice playback begins (after the message is presented, when the audio actually starts). |
| `OnVoiceLineEnded` | The line's voice finishes — naturally, or because the line was skipped/aborted. |

Each comes in a **dynamic** form (for UMG / Blueprint) and a **native** form (for C++ / Slate). Both carry the same payload: the node GUID, the speaker tag, and the sound asset.

```text
StartDialogue
   → OnMessageReceived            (text lands in the UI)
   → OnVoiceLineStarted           ← voice audio begins   ◀── hook lip-sync ON here
        … line plays …
   → OnVoiceLineEnded             ← voice audio ends      ◀── hook lip-sync OFF here
   → (advance)
```

> 📸 **Image placeholder:** `vo-hooks-event-timeline.png` — Timeline graphic of one line's events.
> *Setup:* Create a graphic (not an editor screenshot). A horizontal time arrow. Markers in order: `OnMessageReceived`, `OnVoiceLineStarted` (green), a shaded "voice playing" band, `OnVoiceLineEnded` (red), `advance`. Label the green/red markers "lip-sync ON / OFF". White background.

### Why "AfterVoice" Is Now Robust

The `AfterVoice` advance mode (a SayLine that continues when its voice finishes) is now driven by the same event-based voice-end detection as `OnVoiceLineEnded`, rather than a polling estimate. The result: a line set to `AfterVoice` advances exactly when the audio component reports completion — reliable even for variable-length localized takes.

## No-Code Path (Blueprint)

You can consume the events with zero C++.

1. Get the active `UMayDialogueInstance` (e.g. from `UMayDialogueSubsystem → Get Active Dialogue`, or capture it from the `OnDialogueStarted` event).
2. **Bind Event to On Voice Line Started** and **On Voice Line Ended**.
3. In the bound event, read the payload (`NodeGuid`, `SpeakerTag`, `Sound`) and drive your system — start a lip-sync component, set a mixer state, spawn a subtitle, etc.

> 📸 **Image placeholder:** `vo-hooks-blueprint-bind.png` — Blueprint graph binding both voice-line events on the instance.
> *Setup:* Blueprint event graph. `OnDialogueStarted` → `Bind Event to OnVoiceLineStarted` and `Bind Event to OnVoiceLineEnded` on the instance. Two custom events below: "HandleVoiceStart" (pulls SpeakerTag + Sound) and "HandleVoiceEnd". Show the red event-dispatcher pins wired to the custom events.

{% hint style="warning" %}
**Multiplayer caveat — these events are server-only.** `OnVoiceLineStarted` / `OnVoiceLineEnded` fire on the `UMayDialogueInstance`, which lives only on the **server** (the voice playback and audio component live there too). There is **no** voice-line Client RPC and **no** mirrored voice-line event on `UMayDialogueParticipant` in 1.0 — only message and choice data are mirrored to clients. So on a **dedicated server** these events fire but there is no local audio, and on a **remote client** they do not fire at all.

For a client-side reaction (e.g. lip-sync on a remote-owned character), drive it off a signal that *does* reach the client — the mirrored `OnMessageReceived` (which carries the line's text and emotion tags) — or run an amplitude probe against the client's own playing audio. See the [lip-sync recipe](../recipes/lip-sync-bridge.md) for the honest MP pattern.
{% endhint %}

## API Reference — Voice-Line Delegates

Declared on `UMayDialogueInstance` (module `MayDialogue`):

| Member | Form | Signature (params) |
|---|---|---|
| `OnVoiceLineStarted` | Dynamic (`BlueprintAssignable`) | `(FGuid NodeGuid, FGameplayTag SpeakerTag, USoundBase* Sound)` |
| `OnVoiceLineEnded` | Dynamic (`BlueprintAssignable`) | `(FGuid NodeGuid, FGameplayTag SpeakerTag, USoundBase* Sound)` |
| `OnVoiceLineStartedNative` | Native (C++ / Slate) | `(UMayDialogueInstance*, FGuid, FGameplayTag, USoundBase*)` |
| `OnVoiceLineEndedNative` | Native (C++ / Slate) | `(UMayDialogueInstance*, FGuid, FGameplayTag, USoundBase*)` |

The native variants carry the sender instance as the first parameter (same convention as `OnNodeReachedNative` / `OnEventFiredNative`), so a single subscriber can disambiguate concurrent dialogues.

```cpp
// C++: bind the native voice-start event.
Instance->OnVoiceLineStartedNative.AddWeakLambda(this,
    [this](UMayDialogueInstance* Sender, FGuid NodeGuid,
           FGameplayTag SpeakerTag, USoundBase* Sound)
    {
        StartLipSyncFor(SpeakerTag, Sound);
    });
```

## MetaSound Parameter Binding

When a line's resolved voice asset is a **MetaSound Source**, the plugin pushes per-line parameters into the MetaSound at playback via the audio component's parameter interface (`ISoundParameterControllerInterface`). Your MetaSound graph can read them as input parameters.

| Parameter name | Type | Value |
|---|---|---|
| `MayDialogue.SpeakerTag` | String | The speaking node's `SpeakerTag` as the full hierarchical tag name (e.g. `Dialogue.Speaker.Guard`); empty when unset. |
| `MayDialogue.Emotion` | String | The **first** authored emotion tag (e.g. `Emotion.Angry`); empty when none. |
| `MayDialogue.EmotionTags` | String | **All** emotion tags joined comma-separated (e.g. `Emotion.Angry,Emotion.Loud`); empty when none. |
| `MayDialogue.Pitch` | Float | The resolved pitch multiplier for the line (plugin × speaker × participant × node). |

All four parameter names are **prefixed with `MayDialogue.`** and the tag values are pushed as **String** inputs (the parameter interface exposes no Name setter — a MetaSound graph can convert String→Name itself if it needs a Name pin). This binding is automatic — you do not call anything. Author a MetaSound Source with matching input parameters, assign it as the line's voice (or the speaker's voice), and the values arrive each time the line plays.

### Step by Step — A Pitch/Gain-Reactive MetaSound

1. **Create a MetaSound Source** (`MS_CreatureVoice`). Right-click in the content browser → Sounds → MetaSound Source.
2. **Add input parameters** matching the names above exactly (including the `MayDialogue.` prefix):
   - `MayDialogue.Pitch` (Float, default `1.0`)
   - `MayDialogue.SpeakerTag` (String) — optional, for branching
   - `MayDialogue.Emotion` (String) — optional, the first emotion tag
   - `MayDialogue.EmotionTags` (String) — optional, all emotion tags comma-separated
3. **Wire `MayDialogue.Pitch`** into the pitch shift of your generator (or into a `Multiply` on the wave-player's pitch input). Now the line's resolved pitch multiplier drives the synth.
4. (Optional) **Use `MayDialogue.Emotion`** to pick between two timbres: a `String Equals "Emotion.Angry"` → crossfade gain between a harsh and a soft layer.
5. **Assign `MS_CreatureVoice`** as the `DialogueVoice` on the SayLine (or as the speaker's default voice).
6. **Preview** the line in the [Preview Runner](../editor/preview-runner.md): the synth reacts to the per-line pitch and emotion.

> 📸 **Image placeholder:** `vo-hooks-metasound-graph.png` — A MetaSound Source graph reading the `Pitch` and `EmotionTags` input parameters.
> *Setup:* MetaSound editor open on `MS_CreatureVoice`. Visible: input parameters `Pitch (Float)`, `EmotionTags (String)`, `SpeakerTag (String)` in the Inputs panel. The `Pitch` input wired into a pitch-shift / multiply node feeding the output. A red arrow on the `Pitch` input pin. Clean graph, a handful of nodes.

> 📸 **Image placeholder:** `vo-hooks-sayline-metasound-assigned.png` — SayLine details with a MetaSound Source assigned as the voice.
> *Setup:* SayLine node selected in the dialogue editor. Details panel "SayLine → Audio" section: `DialogueVoice` slot holds `MS_CreatureVoice` (MetaSound icon). `PitchMultiplier = 1.3`, `EmotionTags = Emotion.Angry`. Red arrow on the voice slot.

## Edge Cases & Gotchas

**Parameters only reach MetaSound Sources.** If the voice asset is a plain `SoundWave` or `SoundCue`, the parameter push is a no-op (those formats have no parameter inputs). The `PitchMultiplier` still applies via the normal audio path — only the *named* parameters require a MetaSound graph that declares them.

**Parameter names must match exactly, prefix included.** A MetaSound input named `Pitch` (without the `MayDialogue.` prefix) or `maydialogue.pitch` (wrong casing) will not receive the value. Match the full `MayDialogue.Pitch` / `MayDialogue.SpeakerTag` / `MayDialogue.Emotion` / `MayDialogue.EmotionTags` names from the table above.

**`OnVoiceLineEnded` fires on skip and abort, not just natural end.** If the player skips the line or the dialogue aborts mid-line, the end event still fires so your lip-sync cleanly turns off. Treat it as "the voice for this node is no longer playing", not "the audio reached its last sample".

**Babel does not push MetaSound parameters.** When a line has no voice asset and the Babel synth fills in, the voice-line *events* still fire (so lip-sync sees a "voice"), but there is no MetaSound parameter binding — Babel is its own synthesis path. See [Babel System](babel-system.md).

**No voice asset, Babel disabled → no voice events.** A text-only line with `bEnableBabelVoice` off plays no audio, so `OnVoiceLineStarted` / `OnVoiceLineEnded` do not fire for it. Drive purely-visual reactions off `OnMessageReceived` / `OnNodeReached` instead.

## See Also

- [Lip-Sync via the Bridge](../recipes/lip-sync-bridge.md) — the concrete recipe that consumes these events.
- [Node Overrides](node-overrides.md) — where the per-line pitch multiplier comes from.
- [Babel System](babel-system.md) — the procedural fallback when no voice asset is set.
