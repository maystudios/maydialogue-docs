---
description: How MayDialogue and the Gameplay Ability System work together.
---

# GAS Integration

MayDialogue knows the state of your Gameplay Ability System — and can modify it. Requirements read it, SideEffects write it, all without writing your own C++ code.

## Why GAS in Dialogue?

Dialogue decisions often depend on game state: does the player have the right ability? Is their stamina high enough? Do they currently have a quest tag? Instead of wiring this logic into Blueprint event graphs, you place the condition directly on the dialogue node — as a Requirement on a Choice or Branch.

And when dialogue should trigger a GAS effect (healing, granting a tag, playing a cue), that runs as a SideEffect or Action Node — no glue code needed.

## The Three Integration Pillars

| Pillar | What It Does |
| --- | --- |
| **Requirements** | Check ASC state: tag present? Attribute high enough? Ability granted? |
| **SideEffects / Actions** | Mutate ASC state: add/remove tags, apply GameplayEffect, fire GameplayCue. |
| **Helpers** | Resolve ASCs from the dialogue context — no boilerplate, no manual casts. |

## Module: MayDialogueGAS

All GAS classes live in the separate **`MayDialogueGAS`** module. The core module has no hard GAS dependency. Once `MayDialogueGAS` loads, the GAS classes automatically appear in the node and sub-node pickers in the editor.

{% hint style="info" %}
**No GAS in your project?** The GAS classes can still be loaded, but are ignored when no ASC is found. You can leave the module loaded and simply work with dialogue variables instead of GAS nodes.
{% endhint %}

## Where GAS Intervenes in Dialogue

| Location | Example |
| --- | --- |
| Choice Requirement | *Only if the player has the tag `Story.HeardPassword`* |
| Branch Condition | *If Health < 20 → go to injured variant* |
| SayLine Requirement | *This line only if the NPC has the tag `NPC.IsAngry`* |
| SideEffect on Node | *When entering this SayLine: set a reputation tag* |
| Action Node | *Apply GE_HealMajor to the player* |

## Instigator vs. Target

All GAS classes have a flag that determines which actor is checked or affected:

| Flag Value | Target |
| --- | --- |
| `true` | Instigator — normally the player |
| `false` | Target — normally the NPC |

Depending on the class, the flag is called `bCheckOnInstigator`, `bAddToInstigator`, `bApplyToInstigator`, or `bTriggerOnInstigator` — semantically always the same concept.

> 📸 **Image placeholder:** `gas-overview-table.png` — Overview table in the MayDialogue graph editor.
> *Setup:* Open a dialogue asset with a Choice. The Choice has a `HasTag` Requirement (Instigator = true) and an `AddTag` SideEffect. In the right Details panel both sub-nodes are visible. Annotations show "Requirement checks Instigator ASC" and "SideEffect writes Instigator ASC".

## Quick Example: Password Choice

```text
Choice "I know the password"
  Requirement: HasTag
    RequiredTag: Story.Secret.HeardPassword
    bCheckOnInstigator: true
    bHideOnFail: true          ← completely hide the Choice if tag is missing
```

If the player has the tag (e.g. because they spoke with an informant beforehand), the Choice appears. Otherwise it is invisible.

## Quick Example: Heal Reward as Action Node

```text
[SayLine "Thank you, you saved us."]
  │
  ▼
[ApplyEffect]
  EffectClass: GE_HealMajor
  EffectLevel: 1.0
  bApplyToInstigator: true    ← player is healed
  │
  ▼
[Exit Completed]
```

> 📸 **Image placeholder:** `gas-heal-reward-graph.png` — Fully wired graph with ApplyEffect node.
> *Setup:* Open asset `DA_Reward_Heal`. Visible: `SayLine` node (dark red, text "Thank you…") → `ApplyEffect` node (blue, EffectClass = GE_HealMajor, EffectLevel = 1.0, bApplyToInstigator = true) → `Exit` node (red). Connections horizontal from left to right.

## Chapter Overview

* [Requirements](requirements.md) — `HasTag`, `CheckAttribute`, `HasAbility`
* [Actions](actions.md) — `AddTag`, `RemoveTag`, `ApplyEffect`, `TriggerCue`
* [Helpers](helpers.md) — ASC resolution, context utilities
* [Creating Custom GAS Nodes](extending.md) — Blueprint extension step by step

> 📸 **Image placeholder:** `gas-module-picker.png` — Node picker in the dialogue editor with the GAS category expanded.
> *Setup:* Right-click in the MayDialogue editor graph. Context menu shows category `GAS` with entries `Has Gameplay Tag`, `Check GAS Attribute`, `Has Gameplay Ability`, `Add Gameplay Tag`, `Remove Gameplay Tag`, `Apply Gameplay Effect`, `Trigger Gameplay Cue`. Category expanded, all entries visible.
