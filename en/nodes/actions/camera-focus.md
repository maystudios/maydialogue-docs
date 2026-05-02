---
description: Smoothly pan the camera to a Speaker — with or without a LevelSequence.
---

# Camera Focus

Blends the player camera toward a specific Participant (Speaker or NPC). You can use a manual blend with offset and FOV override, or a LevelSequence for cinematic camera moves.

## When to use

- **Dramatic reveal moment** — Camera pans to the monster just before it speaks.
- **Interrogation scene** — Focus switches per dialogue line from NPC to player and back.
- **Cinematic monologue** — LevelSequence drives a close-up camera move along the narrator.
- **Horror scare moment** — FOV override to 50° + fast blend (0.1 s) for a panic feeling.

---

> 📸 **Image placeholder:** `camera-focus-node.png` — "Camera Focus" Node in the MayDialogue graph.
> *Setup:* Open asset in editor. Node alone visible with clean Input pin on the left, Output pin on the right. Node color blue-grey (camera category). Title bar label: "Camera Focus".

---

## Resolution priority

A single Camera Focus node supports four configurations. The runtime evaluates them top-down and uses the first one that is set:

| Priority | Active property | Result |
|---|---|---|
| 1 | `CameraSequence` | A Level Sequence drives the camera (Cine Camera + Camera Cuts Track). |
| 2 | `CameraAnchorTag` | Finds an `AMayDialogueCameraAnchor` actor in the level by tag and switches the player's ViewTarget to it via `SetViewTargetWithBlend`. |
| 3 | `FocusSpeakerTag` + `ShotTag` | Composes the participant's `ShotAnchors[ShotTag]` with the actor transform, spawns a transient `CineCameraActor` there, and sets it as ViewTarget. |
| 4 | `FocusSpeakerTag` only | Legacy path: rotates the player's `ControlRotation` toward the speaker (no ViewTarget switch). |

Lower-priority paths are skipped once a higher one matches. The original ViewTarget is captured on the first switch and restored on dialogue end (blend time = `DefaultAnchorRestoreBlendTime` from Project Settings).

## Properties

| Property | Type | Description |
|---|---|---|
| `FocusSpeakerTag` | `FGameplayTag` (`Dialogue.Speaker.*`) | The participant referenced by paths 3 and 4. |
| `BlendTime` | `float` | Blend duration in seconds. `-1` = value from Project Settings (`DefaultCameraBlendTime`). |
| `CameraOffset` | `FVector` | Extra world offset (path 4 only — rotate-controller). |
| `FOVOverride` | `float` | FOV in degrees while focused (paths 3 + 4). `0` = no override. Reset on dialogue end. |
| `bShowDialogueText` | `bool` | Shows dialogue text and waits for player advance (behaves like SayLine). Orthogonal to the camera path. |
| `DialogueText` | `FText` | Text shown when `bShowDialogueText = true`. |
| `CameraSequence` | `TSoftObjectPtr<ULevelSequence>` | Path 1 — Level Sequence drives the camera. |
| `bWaitForSequenceEnd` | `bool` | Path 1 — dialogue waits for the sequence's `OnFinished`. |
| `CameraAnchorTag` | `FGameplayTag` (`Dialogue.CameraAnchor.*`) | Path 2 — tag of the level-placed anchor actor. |
| `ShotTag` | `FGameplayTag` (`Dialogue.Shot.*`) | Path 3 — key into `Participant->ShotAnchors`. Requires `FocusSpeakerTag`. |

## Camera Anchor Actor

`AMayDialogueCameraAnchor` is an actor that inherits from `ACineCameraActor`. Place it in your level where you want the camera to live during a shot — the editor shows the cine-camera frustum, focal length, aperture, depth of field, etc. live in the viewport.

| Property | Purpose |
|---|---|
| `AnchorTag` | Identifier (should be unique per level). Referenced from Camera Focus nodes via `CameraAnchorTag`. |
| `BlendTimeOverride` | Per-anchor blend override. Negative = use the node's `BlendTime` / project default. |

Workflow:

1. Drop a `MayDialogueCameraAnchor` actor in the level where the camera should sit.
2. Frame the shot in the viewport (position, rotation, focal length, aperture).
3. Set `AnchorTag` (e.g. `Dialogue.CameraAnchor.MerchantHall.OverShoulder`).
4. In the dialogue asset, set the Camera Focus node's `CameraAnchorTag` to the same tag.

## Shot Anchors on the Participant

`UMayDialogueParticipant::ShotAnchors` is a `TMap<FGameplayTag, FTransform>` that defines a small **per-NPC** "shot vocabulary" (closeup, over-shoulder, wide, …). Transforms are **relative to the participant actor** and travel with it.

Workflow:

1. On the NPC's `MayDialogueParticipant` component, populate `ShotAnchors` with one entry per shot kind (e.g. `Dialogue.Shot.Closeup` → relative transform).
2. In the dialogue asset, set the Camera Focus node's `FocusSpeakerTag` to the speaker and `ShotTag` to the shot.

At runtime a transient `CineCameraActor` is placed at `ShotAnchors[ShotTag] * Participant->GetActorTransform()` and used as the ViewTarget. The transient camera is reused across all Shot focus nodes in the same dialogue and destroyed automatically on dialogue end.

---

> 📸 **Image placeholder:** `camera-focus-details.png` — Details panel of the Camera Focus Node with filled values.
> *Setup:* Select the Node. In the Details panel: `FocusSpeakerTag = Dialogue.Speaker.Guard`, `BlendTime = 0.5`, `CameraOffset = (0, 0, 10)`, `FOVOverride = 0`, `bShowDialogueText = false`, `CameraSequence = empty`.

---

## Action Node or SideEffect Sub-Node?

If the camera pan is the **central dramatic step** of this graph section (the player should consciously notice the camera switching), use the Action Node. If the pan just happens alongside entering a SayLine, attach it as a SideEffect pill to the SayLine.

---

## Example: Focus switch in dialogue

```text
[SayLine: Guard "Halt! Who are you?"]
  │
  ▼
[CameraFocus: FocusSpeakerTag=Dialogue.Speaker.Player, BlendTime=0.4]
  │
  ▼
[SayLine: Player "A friend of the king."]
  │
  ▼
[CameraFocus: FocusSpeakerTag=Dialogue.Speaker.Guard, BlendTime=0.4]
  │
  ▼
[SayLine: Guard "Then pass."]
```

> 📸 **Image placeholder:** `camera-focus-example-graph.png` — Graph snippet of the example above.
> *Setup:* Four Nodes from left to right: SayLine (Guard) → CameraFocus (Player) → SayLine (Player) → CameraFocus (Guard) → SayLine (Guard). Pins connected. Both CameraFocus Nodes visible with `FocusSpeakerTag` in subtitle.

> 📸 **Image placeholder:** `camera-focus-ingame-before-after.png` — Split-screen before and after the pan in the PIE viewport.
> *Setup:* Left: player's perspective on the guard (Guard centered). Right: after CameraFocus on Player — camera shows player's face. Both images with HUD overlay visible.

---

## Pitfalls

{% hint style="warning" %}
**FOV override does not stack.** The original FOV is saved once at the first override. Multiple CameraFocus Nodes with different `FOVOverride` values in sequence do not result in correctly nested restoration. Only use `FOVOverride` once per dialogue or explicitly reset it before the dialogue ends.
{% endhint %}

{% hint style="info" %}
**AutoFocusSpeaker**: In the Project Settings there is `AutoFocusSpeaker`. When active, the camera automatically pans to the current Speaker of a SayLine — then you only need this Node for manual overrides or LevelSequences.
{% endhint %}

- `CameraSequence` runs in parallel — the dialogue advances immediately (unless `bWaitForSequenceEnd = true`).
- `bShowDialogueText` and `bWaitForSequenceEnd = true` simultaneously: `bShowDialogueText` wins (manual advance).
- `FocusSpeakerTag` must correspond to a registered Participant — otherwise no blend, log warning.
- Anchor existence is **not** checked by the validator (anchors live in the level, the validator runs on the asset). A missing anchor logs a runtime `Warning` and the node falls through to the next priority.
- `ShotTag` without `FocusSpeakerTag` is a **validator error** — the shot map cannot resolve without a speaker.
- On a dedicated server (no PlayerController) the anchor / shot paths no-op — correct, since only clients have a ViewTarget.
- Multiple Camera Focus nodes in sequence: the original ViewTarget is captured on the **first** switch, subsequent nodes blend between anchors. On dialogue end the original target is restored.
