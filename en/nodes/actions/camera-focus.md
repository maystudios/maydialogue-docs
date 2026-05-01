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

## Properties

| Property | Type | Description |
|---|---|---|
| `FocusSpeakerTag` | `FGameplayTag` | Tag of the Participant the camera blends to. Must be under `Dialogue.Speaker.*`. |
| `BlendTime` | `float` | Blend duration in seconds. `-1` = value from Project Settings (`DefaultCameraBlendTime`). |
| `CameraOffset` | `FVector` | Additional offset relative to the target Participant's position. |
| `FOVOverride` | `float` | FOV in degrees. `0` = no override. Reset when the dialogue ends. |
| `bShowDialogueText` | `bool` | Shows dialogue text and waits for player advance (behaves like SayLine). |
| `DialogueText` | `FText` | Text shown when `bShowDialogueText = true`. |
| `CameraSequence` | `TSoftObjectPtr<ULevelSequence>` | Optional LevelSequence instead of manual blend. |
| `bWaitForSequenceEnd` | `bool` | Dialogue waits for the sequence to end. Only active when `CameraSequence` is set. |

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
