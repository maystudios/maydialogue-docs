---
description: Drive any lip-sync solution from dialogue voice lines — bind the voice-line events, feed your blendshapes or visemes, clean up on dialogue end.
---

# Bind Lip-Sync (Bridge Recipe)

## Scenario

Your characters should move their mouths while they speak a dialogue line. MayDialogue does **not** ship a lip-sync system — by design, it stays out of facial animation. Instead it gives you precise voice-line events so you can drive *your* lip-sync solution: an audio-amplitude blendshape driver, a viseme-based tool (OVR LipSync, NVIDIA Audio2Face, MetaHuman), or a hand-keyed mouth flap.

This recipe is the **integration pattern**: bind `OnVoiceLineStarted` / `OnVoiceLineEnded`, feed your driver, and tear it down on dialogue end.

## What You Will Learn

- Bind the voice-line lifecycle events on the dialogue instance.
- Feed the line's voice asset into an amplitude-driven or viseme-based lip-sync driver.
- Cleanly stop lip-sync on `OnVoiceLineEnded` and `OnDialogueEnded`.
- Handle the multiplayer case honestly (the voice-line events are server-only).

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- A lip-sync solution already working in isolation (you can make a head talk given a `USoundBase` or an audio amplitude). MayDialogue supplies the *trigger*, not the lip-sync itself.
- The speaker actor has a skeletal mesh with mouth blendshapes or a viseme curve setup.

## The Pattern

```text
[OnDialogueStarted]
   └─▶ resolve the speaking actor, bind voice events
        │
        ├─[OnVoiceLineStarted (NodeGuid, SpeakerTag, Sound)]
        │     └─▶ LIP-SYNC ON: feed Sound (or amplitude) to your driver
        │
        ├─[OnVoiceLineEnded   (NodeGuid, SpeakerTag, Sound)]
        │     └─▶ LIP-SYNC OFF: close the mouth, stop the driver
        │
        └─[OnDialogueEnded]
              └─▶ unbind everything, hard-stop any running lip-sync
```

> 📸 **Image placeholder:** `lip-sync-bridge-flow-diagram.png` — The bind → drive → cleanup flow.
> *Setup:* Create a graphic (not an editor screenshot). Top node `[OnDialogueStarted: bind]`, branching down to three boxes: `[OnVoiceLineStarted → LipSync ON]` (green), `[OnVoiceLineEnded → LipSync OFF]` (red), `[OnDialogueEnded → unbind + hard-stop]` (grey). Vertical layout, white background.

## Step by Step (Blueprint)

### 1. Bind the Events When the Dialogue Starts

In the actor (or a manager) that owns the speaker, bind to the instance's voice events. The instance is available from the `OnDialogueStarted` payload or via `UMayDialogueSubsystem → Get Active Dialogue`.

```text
[OnDialogueStarted]
   │
   ▼
[Bind Event to OnVoiceLineStarted] ── HandleVoiceStart
[Bind Event to OnVoiceLineEnded]   ── HandleVoiceEnd
```

> 📸 **Image placeholder:** `lip-sync-bridge-bp-bind.png` — Blueprint binding the two voice-line events.
> *Setup:* Blueprint event graph. `OnDialogueStarted` red event node → two `Bind Event to ...` nodes for `OnVoiceLineStarted` and `OnVoiceLineEnded`, each wired to a custom event ("HandleVoiceStart", "HandleVoiceEnd"). Clean layout.

### 2. Start Lip-Sync on Voice Line Started

`HandleVoiceStart` receives `(NodeGuid, SpeakerTag, Sound)`:
- Resolve the actor for `SpeakerTag` (e.g. `Get Active Participant Actors` + match, or your own speaker registry).
- Hand the `Sound` to your lip-sync driver, or start your amplitude sampler against the active voice audio.

```text
[HandleVoiceStart (SpeakerTag, Sound)]
   │
   ▼
[Find Speaker Actor (SpeakerTag)]
   │
   ▼
[Start Lip-Sync] ── Mesh: <speaker mesh>, Sound: Sound
```

### 3. Stop Lip-Sync on Voice Line Ended

`HandleVoiceEnd` closes the mouth — set the jaw/mouth blendshape weights back to zero and stop the driver. Because `OnVoiceLineEnded` also fires on skip and abort, this single handler covers natural end, player skip, and mid-line abort.

```text
[HandleVoiceEnd (SpeakerTag)]
   │
   ▼
[Find Speaker Actor (SpeakerTag)]
   │
   ▼
[Stop Lip-Sync] ── Mesh: <speaker mesh>
```

### 4. Clean Up on Dialogue Ended

Bind `OnDialogueEnded` too: unbind the voice events and hard-stop any lip-sync still running (in case a line was cut off). This prevents a mouth that keeps flapping after the conversation closes.

```text
[OnDialogueEnded]
   │
   ▼
[Stop Lip-Sync (all speakers)]
[Unbind voice events]
```

> 📸 **Image placeholder:** `lip-sync-bridge-ingame.png` — PIE screenshot: NPC talking with the dialogue widget showing the line.
> *Setup:* PIE running. Viewport: NPC head mid-speech, mouth open (blendshape driven). Dialogue widget at the bottom shows the current line with typewriter animation. Caption hint: "Mouth driven by OnVoiceLineStarted."

## Concrete Notes Per Lip-Sync Type

### Audio-Amplitude-Driven Blendshapes

The simplest approach: sample the RMS amplitude of the playing voice and map it to a single jaw-open blendshape.

- On `OnVoiceLineStarted`, start a per-tick sampler on the speaker's active audio (a `USoundVisualizationStatics` envelope read, or your own amplitude probe).
- Each tick, set the jaw/mouth-open morph target to the normalized amplitude.
- On `OnVoiceLineEnded`, stop the sampler and lerp the morph back to zero.

This works for every voice asset type — including the **Babel synth** placeholder, since amplitude sampling does not care whether the audio is a recording or synthesized.

### Viseme-Based Tools (OVR LipSync, Audio2Face, MetaHuman)

These tools want the **sound asset** (or a live audio stream) up front to compute viseme/phoneme curves.

- On `OnVoiceLineStarted`, pass the `Sound` payload to the tool's "play and analyze" entry point so it generates the viseme curve in sync with the voice MayDialogue is already playing.
- Drive the speaker's viseme curves from the tool's output.
- On `OnVoiceLineEnded`, stop the tool's playback/analysis and reset the visemes.

{% hint style="warning" %}
**Avoid double playback.** Viseme tools often *also* play the sound. MayDialogue already plays the line's voice. Use the tool in "analyze only" / "drive curves only" mode, or mute the tool's own audio output, so the line is not heard twice.
{% endhint %}

## Multiplayer Note (Read This Before Shipping Co-op)

The voice-line events are **server-only**, and there is **no client mirror for them**. `OnVoiceLineStarted` / `OnVoiceLineEnded` fire on the `UMayDialogueInstance`, which exists only on the server — and in 1.0 the `UMayDialogueParticipant` mirrors only **message** and **choice** data to clients, **not** voice-line lifecycle. There is no voice-line Client RPC to bind on the client.

That means the pattern above (binding `OnVoiceLineStarted` / `OnVoiceLineEnded`) works directly in **Standalone** and on a **listen-server host** — where the driving player is on the server — but **not** on a dedicated server or for a remote co-op client. On a dedicated server those events fire but there is no audio to lip-sync; on a remote client they do not fire at all.

To drive lip-sync on a remote client, use a signal that actually reaches the client:

- **`OnMessageReceived`** *is* mirrored to clients (via the participant's Client RPC). Use it as the "a line is now being spoken" trigger, and start an **amplitude probe** against the client's locally playing voice audio to drive the mouth — this needs no server event and works for Babel placeholder audio too.
- Stop on the next `OnMessageReceived` (the line changed) and on the mirrored dialogue-ended signal.

In short: the `Sound`-payload, voice-line-event path is a **single-player / listen-server-host** convenience. For true multiplayer lip-sync, drive it from the mirrored message + local audio amplitude instead.

## Variations / Going Further

- **Subtitles + lip-sync together**: the same `OnVoiceLineStarted` payload carries the `Sound`; pair it with `OnMessageReceived` (which carries the text) for synchronized captions.
- **Per-speaker drivers**: branch on `SpeakerTag` to route different characters to different lip-sync setups (a MetaHuman hero vs. a simple blendshape NPC).
- **Emotion-aware mouth shapes**: read the line's `EmotionTags` from the message payload to bias the resting mouth pose (a smile for `Emotion.Happy`).

## Troubleshooting

**Mouth never moves.**
The voice events are not bound, or they bind after the first line already played. Bind on `OnDialogueStarted` (which fires before the first line). For a text-only line with Babel disabled, no voice plays, so no voice events fire — use amplitude sampling with Babel enabled, or drive a generic talk loop off `OnMessageReceived` instead.

**Mouth keeps flapping after the dialogue ends.**
You only stopped lip-sync on `OnVoiceLineEnded`, but the last line was skipped/aborted before that fired cleanly. Always also hard-stop on `OnDialogueEnded`.

**Voice is heard twice.**
A viseme tool is playing the sound in addition to MayDialogue. Switch the tool to analyze-only / curve-drive-only mode (see the warning above).

**Wrong character's mouth moves.**
The `SpeakerTag → actor` resolution is matching the wrong actor. Verify your speaker registry, or use `GetActiveParticipantActors` and match on the participant's speaker tag.

## See Also

- [VO Pro Hooks: Voice-Line Events & MetaSound](../audio/vo-hooks.md) — the full event reference these bindings use.
- [NPC Animation During a Line](npc-animation-during-line.md) — body animation alongside the voice line.
- [Bridge & Lifecycle Events](../runtime/bridge-events.md) — the broader external-integration surface.
