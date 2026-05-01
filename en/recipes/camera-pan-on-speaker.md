---
description: Automatically pan the camera to the active speaker – CameraFocus node step by step.
---

# Camera Pan on Speaker

## Scenario

During an interrogation, the camera should pan to the current speaker with each new line: first to the detective, then to the suspect, then back. The **CameraFocus** node handles this — it finds the right actor via the participant tag and blends the camera with a configurable BlendTime.

## What You Will Learn

- Create a CameraFocus node and link it to a participant tag.
- Set BlendTime and CameraOffset.
- Reset the camera automatically at the end of the dialogue.
- Choreograph multiple speaker switches in one dialogue.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- NPC actor has a Participant component with `CameraTargetOffset` set.

## Mini-Graph

```text
[Entry]
   │
   ▼
[CameraFocus: Speaker=Detective  BlendTime=0.5s]
   │
   ▼
[SayLine: Detective – "Where were you on the evening of the 12th?"]
   │
   ▼
[CameraFocus: Speaker=Suspect  BlendTime=0.5s]
   │
   ▼
[SayLine: Suspect – "At home. Alone."]
   │
   ▼
[CameraFocus: Speaker=Detective  BlendTime=0.3s]
   │
   ▼
[SayLine: Detective – "Of course."]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `camera-pan-on-speaker-graph-overview.png` — Interrogation dialogue with alternating CameraFocus nodes.
> *Setup:* Asset `DA_Interrogation_Alibi` open. Left to right: Entry → CameraFocus (gray-blue box, "Detective") → SayLine → CameraFocus ("Suspect") → SayLine → CameraFocus ("Detective") → SayLine → Exit. Nodes visibly alternating between two speaker tags.

## Step by Step

### 1. Configure the Participant CameraTargetOffset

On the Detective actor: **Participant component → CameraTargetOffset** = `{X:0, Y:0, Z:170}` (eye height). Same for the Suspect.

### 2. Insert the CameraFocus Node

From the Entry output → **Create Node → Camera Focus**.

| Property | Value |
|----------|------|
| `SpeakerTag` | `Dialogue.Speaker.Detective` |
| `BlendTime` | `0.5` |
| `CameraOffset` | `{X:-80, Y:30, Z:0}` (slight over-the-shoulder) |
| `bRestoreOnDialogEnd` | `true` |

> 📸 **Image placeholder:** `camera-pan-on-speaker-focus-details.png` — Details panel of the first CameraFocus node.
> *Setup:* First CameraFocus node selected. Details: `SpeakerTag = Dialogue.Speaker.Detective`, `BlendTime = 0.5`, `CameraOffset = (X=-80, Y=30, Z=0)`, `bRestoreOnDialogEnd = true`.

### 3. Build the Dialogue Graph

Pattern: **CameraFocus → SayLine** repeated for each speaker switch. Point the second CameraFocus at `Dialogue.Speaker.Suspect`.

### 4. Third CameraFocus Back to the Detective

Copy the first CameraFocus node (Ctrl+C, Ctrl+V) and set BlendTime to `0.3` for a faster cut back.

### 5. Camera Reset at Dialogue End

`bRestoreOnDialogEnd = true` on the first CameraFocus node → when the Exit node is reached, the original camera position is automatically restored.

### 6. Compile and Test in PIE

In PIE: start the dialogue, the camera pans at each CameraFocus node. After Exit: camera returns to its starting position.

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Interrogation_Alibi
   ├─ Instigator: Get Player Pawn
   └─ Target: Suspect Actor Reference
```

> 📸 **Image placeholder:** `camera-pan-on-speaker-bp-trigger.png` — Blueprint trigger with both actors as participants.
> *Setup:* Player controller BP or Interaction component. `Start Dialogue`: `Instigator = Get Player Pawn`, `Target = Suspect Actor`. Both actors have Participant components.

{% hint style="info" %}
**C++ variant**

```cpp
// Three actors: Player, Detective, Suspect
TArray<AActor*> Participants = { PlayerPawn, DetectiveActor, SuspectActor };
Sub->StartDialogueMulti(Asset, Participants);
```
{% endhint %}

## CameraFocus as a SideEffect

If the camera pan is a secondary step (not the main flow step), attach it as a **SideEffect** on the SayLine:
- SayLine node → SideEffect → CameraFocus.
- The pan happens when the SayLine is entered, not as a separate node in the flow.

## Variations / Going Further

- **FOV override**: `FOVOverride = 70` on the CameraFocus for dramatic wide-angle moments.
- **LevelSequence integration**: for cinematic shots, link the CameraFocus to a LevelSequence.
- **Jump scare**: combine CameraFocus + CameraShake → [Jump Scare with Camera Shake](jump-scare-shake.md).

## Troubleshooting

**Camera doesn't pan.**
Participant component missing on the NPC actor, or `ParticipantTag` doesn't match the `SpeakerTag` in the CameraFocus node.

**Camera jumps instead of blending.**
`BlendTime = 0` or the project's camera system overrides the blend. Check if another Camera Manager is active.

**Camera doesn't return after dialogue.**
`bRestoreOnDialogEnd = false`. Set the flag to `true` on the CameraFocus node, or reset it manually after `OnDialogueEnded`.
