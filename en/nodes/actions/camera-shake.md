---
description: Trigger a camera shake on the player camera — instant, spatial, or global.
---

# Camera Shake

Plays a UE camera shake on the player camera and immediately returns advance. The dialogue continues in parallel — the shake overlays the running dialogue without blocking it.

## When to use

- **Horror jump scare** — A monster appears before the NPC reacts. Shake + SayLine experienced simultaneously.
- **Nearby explosion** — Something detonates during the conversation; the player feels the pressure wave.
- **Spatially limited effect** — NPC stands close to a machine: shake only if the player is close enough (`SpatialRadius`).
- **Dramatic reveal moment** — Strong shake immediately before an important dialogue line is spoken.

---

> 📸 **Image placeholder:** `camera-shake-node.png` — "Camera Shake" Node in the MayDialogue graph.
> *Setup:* Node alone visible, title bar labeled "Camera Shake", blue-grey (camera category), Input pin left / Output pin right. In subtitle: `ShakeClass = BP_ShakeHeavy`, `Scale = 2.0`.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `ShakeClass` | `TSubclassOf<UCameraShakeBase>` | Any UE CameraShake class (Blueprint or C++). |
| `Scale` | `float` | Shake intensity. `1.0` = normal size, `0.0` = no effect. Minimum: 0. |
| `SpatialRadius` | `float` | Effect radius in cm. `0` = global (always). When > 0: shake only plays if the player is close enough to the epicenter. |
| `EpicenterParticipantTag` | `FGameplayTag` | Participant whose position serves as the epicenter. Only active when `SpatialRadius > 0`. |
| `FalloffInnerRadius` | `float` | Full intensity within this radius. Between inner and outer: linear falloff. Only active when `SpatialRadius > 0`. |

---

> 📸 **Image placeholder:** `camera-shake-details.png` — Details panel with spatial setup.
> *Setup:* Select the Node. In the Details panel: `ShakeClass = BP_ShakeMedium`, `Scale = 1.5`, `SpatialRadius = 500`, `EpicenterParticipantTag = Dialogue.Speaker.Monster`, `FalloffInnerRadius = 200`.

---

## Action Node or SideEffect Sub-Node?

If the shake is **the dramatic main point** of this step (e.g. the scare moment), use the Action Node. If the shake is just a subtle accompanying effect of a SayLine (slight rumble while speaking), attach it as a SideEffect pill to the SayLine.

---

## Example: Jump scare sequence

```text
[SayLine: Player "I can't hear anything anymore..."]
  │
  ▼
[CameraShake: ShakeClass=BP_ShakeHeavy, Scale=2.0]
  │
  ▼
[SayLine: Monster "NOW I AM HERE."]
```

> 📸 **Image placeholder:** `camera-shake-example-graph.png` — Graph snippet of the above sequence.
> *Setup:* Three Nodes from left to right: SayLine (Player) → CameraShake → SayLine (Monster). All pins connected. CameraShake Node shows in subtitle: `Scale = 2.0`.

> 📸 **Image placeholder:** `camera-shake-ingame-moment.png` — Screenshot of running PIE at the moment of the shake.
> *Note for screenshot:* Start PIE, advance to the CameraShake step. Image shows a slightly jolted/offset camera view with the HUD — visible "shake" artifact on the frame. Optional: before/after frame side by side.

---

## Pitfalls

{% hint style="info" %}
The Node is **always pass-through** — no `bWaitForShakeEnd`. If you want a SayLine after the shake that only appears once the camera has settled, add a short `Wait` Node in between.
{% endhint %}

- `SpatialRadius = 0` means global shake (no spatial falloff, always plays).
- `EpicenterParticipantTag` must correspond to a registered Participant — otherwise falls back to the Instigator position.
- Shake and CameraFocus overlap without conflicts — the blend continues, the shake lays on top.
