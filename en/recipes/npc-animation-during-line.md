---
description: NPC nods, gestures, or points at something while a SayLine plays – PlayAnimation node.
---

# NPC Animation During Line

## Scenario

An NPC nods in agreement while making a statement. Another points at a map on the wall while explaining. These gestures happen simultaneously with the SayLine — no extra dialogue step, no Blueprint code. The **PlayAnimation** node plays a montage on the NPC actor in parallel with the text flow.

## What You Will Learn

- Link a PlayAnimation node to a gesture montage.
- `bWaitForEnd = false` for parallel execution vs. `true` for sequential.
- Choreograph multiple animations in one dialogue.
- Use `AdvanceMode = AfterAnimation` on a SayLine.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- Montage assets available: `AM_NPC_Nod`, `AM_NPC_PointRight`.
- NPC actor has an AnimInstance with the `DefaultSlot` slot active.

## Mini-Graph – Parallel (Gesture + Line at the Same Time)

```text
[Entry]
   │
   ▼
[PlayAnimation: NPC – AM_NPC_Nod  WaitForEnd: false]
   │
   ▼
[SayLine: NPC – "Yes, exactly like that."  AdvanceMode: AfterVoice]
   │
   ▼
[PlayAnimation: NPC – AM_NPC_PointRight  WaitForEnd: false]
   │
   ▼
[SayLine: NPC – "Over there, behind the tower."  AdvanceMode: AfterVoice]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `npc-animation-during-line-graph-overview.png` — Dialogue graph with two PlayAnimation nodes before the SayLines.
> *Setup:* Asset `DA_Guide_Explanation` open. Entry → PlayAnimation "Nod" (orange box) → SayLine → PlayAnimation "PointRight" (orange box) → SayLine → Exit. `bWaitForEnd = false` visible on both PlayAnimation nodes in the Details panel.

## Step by Step

### 1. Insert the PlayAnimation Node

From the Entry output → **Create Node → Play Animation**:

| Property | Value |
|----------|------|
| `TargetParticipantTag` | `Dialogue.Speaker.Guide` |
| `Montage` | `AM_NPC_Nod` |
| `StartSection` | `Default` |
| `bWaitForEnd` | `false` |

`bWaitForEnd = false` → the dialogue flow immediately moves to the next node; the montage runs in parallel.

> 📸 **Image placeholder:** `npc-animation-during-line-anim-details.png` — Details panel of the PlayAnimation node.
> *Setup:* PlayAnimation node selected. Details: `TargetParticipantTag = Dialogue.Speaker.Guide`, `Montage = AM_NPC_Nod`, `StartSection = "Default"`, `bWaitForEnd = false (checkbox empty)`.

### 2. SayLine with AfterVoice

From the PlayAnimation output → SayLine *"Yes, exactly like that."*:
- `AdvanceModeOverride = AfterVoice` → the line ends when the voice finishes, not when the player clicks.

This way the gesture runs exactly during the voice playback.

### 3. Second Gesture Animation

From the SayLine output → second PlayAnimation node (`AM_NPC_PointRight`, `bWaitForEnd = false`) → second SayLine.

### 4. Ensure the Montage Slot

For the PlayAnimation node to work, the NPC's AnimInstance must have an animation slot named `DefaultSlot`. Check this in the AnimInstance Blueprint under *Asset Details → Montage Slots*.

### 5. Compile and Test in PIE

In PIE: start the dialogue. Each PlayAnimation node plays the montage on the NPC while the SayLine runs. No click needed between gesture and line.

## bWaitForEnd = true – Sequential

If the SayLine should only start after the animation:

```text
[PlayAnimation: AM_NPC_ThinkingPose  WaitForEnd: true]
   │
   ▼
[SayLine: "Hmm... I think..."]
```

With `bWaitForEnd = true`, the dialogue flow stops at PlayAnimation and waits for the montage to end. Only then does the SayLine follow. Good for dramatic pauses.

## AdvanceMode = AfterAnimation on SayLine

As an alternative to PlayAnimation as its own node: set `AdvanceModeOverride = AfterAnimation` directly on the SayLine. The SayLine waits for the end of a running montage. Prerequisite: a montage must already be playing (e.g. started by a preceding PlayAnimation node).

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Guide_Explanation
   └─ ...
```

> 📸 **Image placeholder:** `npc-animation-during-line-ingame.png` — PIE screenshot: NPC nodding, dialogue widget showing the line at the same time.
> *Setup:* PIE running. In the viewport: NPC character in the Nod pose. Dialogue widget at the bottom shows `"Yes, exactly like that."` with typewriter animation. Both visible simultaneously.

## Variations / Going Further

- Player animation instead of NPC: `TargetParticipantTag = Dialogue.Participant.Player` — the player character performs a reaction animation.
- Combine jump-scare montage + CameraShake → [Jump Scare with Camera Shake](jump-scare-shake.md).
- Choose animation based on a quest variable: a Branch before PlayAnimation that selects `AM_Friendly` or `AM_Aggressive` depending on the variable.

## Troubleshooting

**Animation doesn't play.**
Participant tag doesn't match the NPC actor tag. NPC AnimInstance doesn't have `DefaultSlot`. Montage is not exported for `DefaultSlot`.

**Animation ends immediately.**
Montage length is very short, or `EndSection` in the montage asset jumps directly to the end. Check the montage in the AnimInstance preview.

**Dialogue flow stops at PlayAnimation even though bWaitForEnd = false.**
`bWaitForEnd` is set to `true` in the Details panel. Check the checkbox state.
