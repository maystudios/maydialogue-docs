---
description: A scary moment in dialogue with camera shake and NPC animation – atmospheric horror without a cutscene.
---

# Jump Scare with Camera Shake

## Scenario

Mid-conversation, the NPC makes an aggressive move and the camera shakes. The player gets a fright without needing a cutscene. This recipe combines a **CameraShake** node, a **PlayAnimation** node, and a short sound for maximum effect.

## What You Will Learn

- Configure a CameraShake node with a `UCameraShakeBase` subclass.
- Use a PlayAnimation node for the NPC's scare montage.
- Use a PlaySound node for a non-voice stinger.
- The timing order: animation first, then shake, then a reaction SayLine.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- CameraShake asset available (Blueprint subclass of `UCameraShakeBase`).
- Montage asset for the NPC's scare movement.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: NPC – "You should..." AdvanceMode: Manual]
   │
   ▼
[PlayAnimation: NPC – Montage: AM_NPC_LungeForward  WaitForEnd: false]
   │
   ▼
[CameraShake: BS_HorrorShake  Scale: 1.5  Radius: 500cm]
   │
   ▼
[PlaySound: SE_JumpScareStinger  2D: true]
   │
   ▼
[SayLine: NPC – "...NEVER come back!"]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `jump-scare-shake-graph-overview.png` — Horror dialogue graph with three action nodes for the scare moment.
> *Setup:* Asset `DA_Horror_NPC_Threat` open. SayLine → PlayAnimation (orange box) → CameraShake (red box) → PlaySound (yellow box) → SayLine → Exit. All three nodes prominently in the main flow — easy to see what makes up the scare moment.

## Step by Step

### 1. Create the CameraShake Asset

In UE: new Blueprint with parent `UCameraShakeBase` (e.g. `WaveOscillatorCameraShake`). Example horror settings:

| Property | Value |
|----------|------|
| `OscillationDuration` | `0.5` |
| `RotOscillation.Pitch.Amplitude` | `3.0` |
| `RotOscillation.Yaw.Amplitude` | `2.0` |
| `LocOscillation.Z.Amplitude` | `5.0` |

Asset name: `BS_HorrorShake`.

### 2. Configure the PlayAnimation Node

From the SayLine output → **Create Node → Play Animation**:

| Property | Value |
|----------|------|
| `TargetParticipantTag` | `Dialogue.Speaker.HorrorNPC` |
| `Montage` | `AM_NPC_LungeForward` |
| `StartSection` | `Default` |
| `bWaitForEnd` | `false` (dialogue flow continues immediately, animation runs in parallel) |

> 📸 **Image placeholder:** `jump-scare-shake-animation-details.png` — Details panel of the PlayAnimation node.
> *Setup:* PlayAnimation node selected. Details: `TargetParticipantTag = Dialogue.Speaker.HorrorNPC`, `Montage = AM_NPC_LungeForward`, `bWaitForEnd = false`.

### 3. Configure the CameraShake Node

From the PlayAnimation output → **Create Node → Camera Shake**:

| Property | Value |
|----------|------|
| `ShakeClass` | `BS_HorrorShake` |
| `Scale` | `1.5` |
| `PlaySpace` | `CameraLocal` |
| `bAttenuateWithRadius` | `true` |
| `Radius` | `500.0` |
| `TargetActor` | *(empty = always play)* |

### 4. PlaySound Node for the Stinger

From the CameraShake output → **Create Node → Play Sound**:

| Property | Value |
|----------|------|
| `Sound` | `SE_JumpScareStinger` |
| `b2DSound` | `true` |
| `VolumeMultiplier` | `1.2` |

`b2DSound = true` — the stinger doesn't come from the room, but directly in the mix.

> 📸 **Image placeholder:** `jump-scare-shake-playsound-details.png` — PlaySound node with 2D flag.
> *Setup:* PlaySound node selected. Details: `Sound = SE_JumpScareStinger`, `b2DSound = true (checkbox)`, `VolumeMultiplier = 1.2`.

### 5. Reaction SayLine

From the PlaySound output → SayLine *"...NEVER come back!"* → Exit.

### 6. Compile and Test in PIE

In PIE: start the dialogue. At the second SayLine, the montage should play, the camera shake, and the stinger fire — nearly simultaneously due to the short node execution time.

## Timing Optimization

If the shake and sound come a frame too early:
- Insert a SayLine before the shake with `AdvanceMode = Timer` and `AutoAdvanceDelay = 0.1` (micro-pause before shake).
- Or use PlayAnimation with `bWaitForEnd = true` → shake fires only when the montage finishes.

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Horror_NPC_Threat
   └─ ...
```

## Variations / Going Further

- Camera pan BEFORE the shake: [Camera Pan on Speaker](camera-pan-on-speaker.md) + then shake.
- Combine NPC animation with a player reaction → [NPC Animation During Line](npc-animation-during-line.md).
- Shake via GameplayCue instead of direct node: `TriggerCue` node → GC_HorrorShake.

## Troubleshooting

**CameraShake doesn't happen.**
The player camera is not a `APlayerCameraManager`-supported camera system. Check if `GetPlayerCameraManager()` returns a valid value. With custom camera logic: trigger CameraShake manually via `PlayCameraShake` in the OnDialogueEvent handler.

**Animation plays, but shake doesn't happen.**
`bWaitForEnd = true` on the PlayAnimation node and the montage is very long. Set `bWaitForEnd = false`.

**Stinger is too loud.**
Reduce `VolumeMultiplier` or configure a ducking channel for dialogue stingers in the Sound Mix.
