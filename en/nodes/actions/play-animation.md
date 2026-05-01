---
description: Play an AnimMontage on a dialogue Participant — with or without waiting.
---

# Play Animation

Plays a `UAnimMontage` on a Participant. You can choose the playback speed and a start section. Optionally, the dialogue waits for the montage to end before continuing.

## When to use

- **NPC gesture on agreement** — Head nod directly after the player chose "I'll help you."
- **Attack animation before combat** — NPC draws weapon and waits for the montage to end before the dialogue exits.
- **Surprise reaction** — NPC recoils (fire-and-forget, no waiting), dialogue continues immediately.
- **Ritual scene** — NPC performs a long summoning animation, dialogue text only appears afterward.

---

> 📸 **Image placeholder:** `play-animation-node.png` — "Play Animation" Node in the MayDialogue graph.
> *Setup:* Node alone, title bar "Play Animation" (category color: orange/animation). Subtitle shows: `AnimationTargetTag = Dialogue.Speaker.NPC`, `Montage = AM_Nod`. Input pin left, Output pin right.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `Montage` | `UAnimMontage*` | The montage asset to play. |
| `AnimationTargetTag` | `FGameplayTag` | Tag of the Participant whose AnimInstance receives the montage. |
| `PlayRate` | `float` | Playback speed. `1.0` = normal. Minimum: 0.01. |
| `StartSection` | `FName` | Start section within the montage. Empty = from the beginning. |
| `bWaitForMontageEnd` | `bool` | When `true`: dialogue pauses until `OnMontageEnded` fires. When `false`: fire-and-forget. |

---

> 📸 **Image placeholder:** `play-animation-details.png` — Details panel with filled values.
> *Setup:* Select the Node. In the Details panel: `Montage = AM_GuardReact`, `AnimationTargetTag = Dialogue.Speaker.Guard`, `PlayRate = 1.0`, `StartSection = (empty)`, `bWaitForMontageEnd = true`.

> 📸 **Image placeholder:** `play-animation-ingame-npc.png` — NPC in the montage pose in the PIE viewport.
> *Setup:* Start PIE, run through the dialogue to the PlayAnimation step. Screenshot of the NPC during the montage (head-nod pose or dodge pose). Camera should show the NPC, HUD visible with active dialogue text.

---

## Action Node or SideEffect Sub-Node?

If the animation is **the main point of the step** (e.g. NPC nods solemnly in response to a choice), use the Action Node. If the animation is just an accompanying effect of a SayLine (NPC gestures while speaking), attach it as a SideEffect pill to the SayLine.

---

## Example: Gesture + reaction line

```text
[PlayerChoice: "I will help you."]
  │
  ▼
[PlayAnimation: AM_NodAgreement, Target=NPC, bWaitForMontageEnd=false]
  │
  ▼
[SayLine: NPC "Good. Then hurry."]
```

For a waiting example (ritual scene):

```text
[SayLine: NPC "Watch carefully."]
  │
  ▼
[PlayAnimation: AM_Ritual, Target=Shaman, bWaitForMontageEnd=true]
  │
  ▼
[SayLine: NPC "It is done."]
```

> 📸 **Image placeholder:** `play-animation-example-graph.png` — Graph snippet of the ritual example.
> *Setup:* Three Nodes from left to right: SayLine (NPC) → PlayAnimation (bWaitForMontageEnd=true visible in subtitle) → SayLine (NPC). Pins connected.

---

## Pitfalls

{% hint style="warning" %}
The validator warns if `bWaitForMontageEnd=true` is set but no Output pin is connected. In that case the dialogue would wait forever.
{% endhint %}

- `AnimationTargetTag` must correspond to a registered Participant — otherwise no play, log warning.
- `StartSection` must be a valid section name of the montage — otherwise the montage plays from the beginning.
