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

## Resolution priority — which path runs

A Camera Focus node supports three configurations. The runtime evaluates them top-down and uses the first one that is set:

| Priority | Active property | Result |
|---|---|---|
| 1 | `CameraSequence` | A Level Sequence drives the camera (Cine Camera + Camera Cuts Track). |
| 2 | `CameraAnchorTag` | Finds an `AMayDialogueCameraAnchor` by tag and switches the player's ViewTarget to it via `SetViewTargetWithBlend`. |
| 3 | `FocusSpeakerTag` only | Legacy path: smoothly rotates the player's `ControlRotation` toward the speaker. **No ViewTarget switch** — the real pawn camera (spring-arm, lag, post-process) stays active. |

Lower-priority paths are skipped once a higher one matches. The original ViewTarget is captured on the first switch and restored on dialogue end (blend time = `DefaultAnchorRestoreBlendTime` from Project Settings).

## Override priority — who wins per value

Every value follows the same rule: **most-specific wins**, with sentinel values meaning "not set, fall through".

| Value | Source 1 (priority) | Source 2 | Fallback |
|---|---|---|---|
| **BlendTime (Anchor path)** | Node `BlendTime` ≥ 0 | Anchor `BlendTimeOverride` ≥ 0 | Settings `DefaultCameraBlendTime` |
| **BlendTime (Legacy path)** | Node `BlendTime` ≥ 0 | — | Settings `DefaultCameraBlendTime` |
| **Restore BlendTime** | — | — | Settings `DefaultAnchorRestoreBlendTime` |
| **FOV (Legacy path)** | Node `FOVOverride` > 0 | — | Current `PlayerCameraManager` FOV |
| **FOV (Anchor path)** | — | — | The anchor's CineCamera settings (focal length, aperture, …) |

### Look-at position (Legacy path only)

The one place that is **additive** rather than override:

```
LookAt = SpeakerActor.Location
       + (Node.bIgnoreParticipantOffset ? 0 : Participant.CameraTargetOffset)
       + Node.CameraOffset
```

`Participant.CameraTargetOffset` is set **once per NPC** (e.g. "head height = +60 cm"), `Node.CameraOffset` is per-scene fine-tuning. Adding them is useful default behavior — it avoids forcing every node to repeat the per-NPC base offset. Set `bIgnoreParticipantOffset = true` on the node when a special-case shot should not inherit the participant's offset.

In the Anchor path these offsets have **no effect** — the anchor actor defines its pose entirely on its own.

## Properties

| Property | Type | Description |
|---|---|---|
| `FocusSpeakerTag` | `FGameplayTag` (`Dialogue.Speaker.*`) | Speaker for path 3. |
| `BlendTime` | `float` | Blend duration in seconds. `-1` = value from Project Settings (`DefaultCameraBlendTime`). |
| `CameraOffset` | `FVector` | Extra world offset (path 3 only, additive with `Participant.CameraTargetOffset`). |
| `bIgnoreParticipantOffset` | `bool` | If `true`, ignore `Participant.CameraTargetOffset` and use only this node's `CameraOffset`. Default: `false`. |
| `bLockLookInputDuringBlend` | `bool` | Legacy path: suppress player look input during the blend so the lerp is not fought by mouse/stick. No effect on Anchor path. Default: `true`. |
| `FOVOverride` | `float` | FOV in degrees while focused (path 3 only). `0` = no override. |
| `bShowDialogueText` | `bool` | Shows dialogue text and waits for player advance (behaves like SayLine). Orthogonal to the camera path. |
| `DialogueText` | `FText` | Text shown when `bShowDialogueText = true`. |
| `CameraSequence` | `TSoftObjectPtr<ULevelSequence>` | Path 1 — Level Sequence drives the camera. |
| `bWaitForSequenceEnd` | `bool` | Path 1 — dialogue waits for the sequence's `OnFinished`. |
| `CameraAnchorTag` | `FGameplayTag` (`Dialogue.CameraAnchor.*`) | Path 2 — tag of the anchor actor. |

## Camera Anchor Actor — the core tool for staged shots

`AMayDialogueCameraAnchor` inherits from `ACineCameraActor`. Place it in your level — the editor shows the cine-camera frustum, focal length, aperture, depth of field etc. live in the viewport. Designers frame the shot by viewport drag, not by typing vector values.

| Property | Purpose |
|---|---|
| `AnchorTag` | Identifier (should be unique per level). Referenced from Camera Focus nodes via `CameraAnchorTag`. |
| `BlendTimeOverride` | Per-anchor blend override. Negative = use the node's `BlendTime` / project default. |

### World-anchored shot

Drop the anchor loose in the level — it stays put no matter where the NPC moves. Best for set pieces (throne room, market stall, cutscene camera point).

### Character-anchored shot — parent the anchor to the NPC

If the shot should **travel with the NPC** (closeup, over-shoulder, …):

1. Drop a `MayDialogueCameraAnchor` actor in the level.
2. In the **World Outliner**, drag the anchor onto the NPC actor → it becomes a **child actor** whose transform is now relative to the NPC.
3. Frame the shot in the viewport — the anchor follows the NPC wherever it moves.
4. Set `AnchorTag`, reference it from the dialogue asset.

That's the entire mechanism for "NPC-relative shots": native UE actor parenting, no separate concept. The advantage over invisible transform maps: **everything is visible in the viewport, draggable, with live cine-camera preview**.

Tip: for NPC blueprints, define the anchors as child-actor components directly in the blueprint — every NPC spawn then ships with its shot vocabulary built-in.

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
- On a dedicated server (no PlayerController) the anchor path no-ops — correct, since only clients have a ViewTarget.
- Multiple Camera Focus nodes in sequence: the original ViewTarget is captured on the **first** switch, subsequent nodes blend between anchors. On dialogue end the original target is restored.
