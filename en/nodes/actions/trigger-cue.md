---
description: Fire a GameplayCue one-shot — particles, SFX, UI flash, combined and replicated.
---

# Trigger Cue

Fires a `GameplayCue` as a one-shot via the `AbilitySystemComponent`. The cue runs cosmetically — particles, sound, UI effect — replicated via GAS to all clients. Pass-through, no waiting.

## When to use

- **Curse visual effect** — NPC speaks a curse, particles + sound + UI flash appear simultaneously on the player.
- **Healing glow** — Healer lays on hands, `GameplayCue.Dialog.Heal` flashes green on the NPC.
- **Magic hit** — Wizard strikes the air, cue hits the player (`bTriggerOnInstigator=true`).
- **Multiplayer-safe cosmetic** — Effect must be visible on all clients simultaneously (no PlaySound workaround needed).

---

> 📸 **Image placeholder:** `trigger-cue-node.png` — "Trigger Cue" Node in the MayDialogue graph.
> *Setup:* Node alone, title bar "Trigger Cue" (category color: purple/GAS). Subtitle shows: `GameplayCue.Dialog.Curse → Target`. Input and Output pins visible.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `CueTag` | `FGameplayTag` | Tag of the GameplayCue. Must be under `GameplayCue.*`. |
| `bTriggerOnInstigator` | `bool` | `true` = cue runs on the player's ASC. `false` = cue runs on the target NPC's ASC. Default: `true`. |

---

> 📸 **Image placeholder:** `trigger-cue-details.png` — Details panel with NPC target setup.
> *Setup:* Select the Node. In the Details panel: `CueTag = GameplayCue.Dialog.Curse`, `bTriggerOnInstigator = false`.

---

## Action Node or SideEffect Sub-Node?

If the cue effect is the **dramatic main moment** of this graph step (the curse moment is intentionally staged as its own step), use the Action Node. If the cue is just a silent cosmetic side effect of a SayLine, attach it as a SideEffect pill to the SayLine.

---

## TriggerCue vs. PlaySound vs. CameraShake

| | Trigger Cue | Play Sound | Camera Shake |
|---|---|---|---|
| System | GAS (`ExecuteGameplayCue`) | UE Audio direct | UE Camera |
| Replicated | Yes, via GAS | No | Yes (`ClientStartCameraShake`) |
| Typical content | Particles + Sound + UI combined | Sound only | Camera shake only |
| Runs on | Target actor / Instigator | World position or 2D | Player camera |

If you only need a sound: [Play Sound](play-sound.md). If you only need a camera shake: [Camera Shake](camera-shake.md). If you need particles + sound + UI flash combined and replicated: Trigger Cue.

---

## Example: Curse scene

```text
[SayLine: NPC "I curse you!"]
  │
  ▼
[TriggerCue: CueTag=GameplayCue.Dialog.Curse, bTriggerOnInstigator=false]
  │
  ▼
[SayLine: NPC "May the gods no longer follow you."]
```

> 📸 **Image placeholder:** `trigger-cue-example-graph.png` — Graph snippet of the curse scene.
> *Setup:* Three Nodes: SayLine (NPC) → TriggerCue (GameplayCue.Dialog.Curse, "→ Target" in subtitle) → SayLine (NPC). All pins connected.

---

## Pitfalls

{% hint style="warning" %}
`CueTag` must be under `GameplayCue.*` — other tag hierarchies are not recognized by GAS routing. A wrong prefix results in a silent no-op without an error.
{% endhint %}

{% hint style="info" %}
For **one-shot effects**, use `GameplayCueNotify_Static` as the cue class (no actor, no persistent state). Persistent cues (`GameplayCueNotify_Actor`) must be managed explicitly via the `Add`/`Remove` mechanism — Trigger Cue alone is not sufficient for that.
{% endhint %}

- The cue runs at the **target actor's origin**, not in camera space — particles appear on the NPC, not in front of the camera.
- This action is part of the GAS integration and usable without additional setup. It accesses the Gameplay Ability System — make sure your characters have an `AbilitySystemComponent`.
- The target must have a `UAbilitySystemComponent` — otherwise no-op + log warning.
