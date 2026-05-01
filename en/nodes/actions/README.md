---
description: Overview of all Action Nodes and when to choose which.
---

# Action Nodes

Action Nodes are visible, standalone steps in the dialogue graph. Each has an Input and an Output pin, can be paused at a breakpoint, and can be commented in the Outline. Use them when **this action is the main point of this step**.

> **Action Node or SideEffect Sub-Node?**
> If the action structurally shapes the flow or should be debuggable in its own right, use the Action Node as its own box. If it just happens alongside a SayLine (e.g. a sound effect when entering a line), attach it as a SideEffect pill to the relevant Node.

---

## All Action Nodes at a Glance

| Node | Category | What it does |
|---|---|---|
| [Camera Focus](camera-focus.md) | Camera | Smoothly blends the player camera toward a Speaker. |
| [Camera Shake](camera-shake.md) | Camera | Plays a camera shake on the player camera. |
| [Play Animation](play-animation.md) | Animation | Plays a montage on a Participant — optionally with waiting. |
| [Apply Effect](apply-effect.md) | GAS | Applies a `UGameplayEffect` to the player or a target. |
| [Set Variable](set-variable.md) | Data | Writes a dialogue variable (Bool / Int / Float / String / Tag). |
| [Fire Event](fire-event.md) | Data | Fires a GameplayTag event to external systems. |
| [Play Sound](play-sound.md) | Audio | Plays a non-voice sound (2D or 3D). |
| [Add Tag](add-tag.md) | GAS | Sets a LooseGameplayTag on an ASC. |
| [Remove Tag](remove-tag.md) | GAS | Removes a LooseGameplayTag from an ASC. |
| [Trigger Cue](trigger-cue.md) | GAS | Fires a GameplayCue one-shot (particles, SFX, UI flash). |

---

## Shared Rules

- All Action Nodes have **one Input and one Output pin**.
- Most are **pass-through** — the dialogue continues immediately after execution.
- Exceptions: `Play Animation` with `bWaitForMontageEnd = true` pauses the dialogue until the montage ends. `Camera Focus` with `bWaitForSequenceEnd = true` waits for the Level Sequence.
- Breakpoints on Action Nodes pause execution before the Node — useful when debugging complex flows.
