---
description: AddTag, RemoveTag, ApplyEffect, TriggerCue — as Action Node or SideEffect.
---

# GAS Actions

Four predefined actions cover the most common GAS mutations in dialogue. Each exists in two forms — the logic is identical, only the visual placement in the graph differs.

| Form | When to Use |
| --- | --- |
| **Action Node** (its own box in the graph) | The action is the main step of this section of the graph |
| **SideEffect sub-node** (pill on another node) | The action happens as a side effect when entering a node |

---

## Add Gameplay Tag

**Class:** `UMayDlgSideEffect_AddTag`

Adds a loose GameplayTag to the ASC. The tag persists until explicitly removed — it is not bound to a GameplayEffect.

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `TagToAdd` | `FGameplayTag` | The tag to add. |
| `bAddToInstigator` | bool | `true` = player ASC; `false` = NPC ASC. |

> 📸 **Image placeholder:** `action-addtag-details.png` — Details panel of the AddTag sub-node.
> *Setup:* Open SayLine node, expand SideEffect array, select `AddTag` sub-node. Details panel shows: `TagToAdd = Story.Accepted.QuestHelp`, `bAddToInstigator = true`.

### Example: Setting a Story Flag

```text
SayLine "Of course I'll help you."
  SideEffects:
    + Add Gameplay Tag
        TagToAdd:       Story.Accepted.QuestHelp
        bAddToInstigator: true
```

From this moment on, the player ASC carries the tag — other dialogues can check for it.

---

## Remove Gameplay Tag

**Class:** `UMayDlgSideEffect_RemoveTag`

Removes a loose GameplayTag from the ASC. Only works for tags set via `AddLooseGameplayTag`, not for tags from active GameplayEffects.

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `TagToRemove` | `FGameplayTag` | The tag to remove. |
| `bRemoveFromInstigator` | bool | `true` = player ASC; `false` = NPC ASC. |

### Example: Removing a Disguise

```text
SayLine "I recognize you — drop the disguise!"
  SideEffects:
    + Remove Gameplay Tag
        TagToRemove:         Story.Disguised
        bRemoveFromInstigator: true
```

---

## Apply Gameplay Effect

**Class:** `UMayDlgSideEffect_ApplyEffect`

Applies a `UGameplayEffect` to an ASC. The Source ASC is always the Instigator. The Target ASC is determined by the flag.

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `EffectClass` | `TSubclassOf<UGameplayEffect>` | The effect class to apply. |
| `EffectLevel` | float | Effect level (>= 0, default 1.0). |
| `bApplyToInstigator` | bool | `true` = apply to player; `false` = apply to NPC. |

> 📸 **Image placeholder:** `action-applyeffect-details.png` — Details panel of the ApplyEffect node.
> *Setup:* Select ApplyEffect Action Node in the graph. Details panel shows: `EffectClass = GE_HealMajor`, `EffectLevel = 1.0`, `bApplyToInstigator = true`. EffectClass picker shows the Blueprint name of the effect.

### As Action Node (Main Step)

```text
[SayLine "For your loyalty — take this blessing."]
  │
  ▼
[Apply Gameplay Effect]
  EffectClass: GE_BlessingBuff
  EffectLevel: 1.0
  bApplyToInstigator: true
  │
  ▼
[Exit Completed]
```

### As SideEffect (Side Step)

```text
SayLine "A small token of thanks..."
  SideEffects:
    + Apply Gameplay Effect
        EffectClass:       GE_SmallHeal
        EffectLevel:       1.0
        bApplyToInstigator: true
```

{% hint style="warning" %}
**Applying to the NPC:** Set `bApplyToInstigator = false` to apply the effect to the NPC. The Source ASC is always the player — this is the standard GAS convention for "player heals NPC".
{% endhint %}

---

## Trigger Gameplay Cue

**Class:** `UMayDlgSideEffect_TriggerCue`

Fires a GameplayCue on an ASC. Supports three modes: one-shot Execute, persistent Add, and Remove.

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `CueTag` | `FGameplayTag` | Must be under the `GameplayCue.*` hierarchy. |
| `bTriggerOnInstigator` | bool | `true` = player ASC; `false` = NPC ASC. |
| `Mode` | `EMayDlgCueMode` | `Execute` (one-shot) / `Add` (persistent) / `Remove` |

| Mode | When to Use |
| --- | --- |
| `Execute` | Short visual or audio effect, e.g. a flash or a chime. |
| `Add` | Persistent visual state, e.g. a halo that stays until `Remove`. |
| `Remove` | Stop a cue previously started with `Add`. |

> 📸 **Image placeholder:** `action-triggercue-details.png` — Details panel of the TriggerCue sub-node.
> *Setup:* Select TriggerCue SideEffect on a SayLine node. Details panel shows: `CueTag = GameplayCue.UI.BuffFlash`, `bTriggerOnInstigator = true`, `Mode = Execute (One-Shot)`.

### Example: Visual Feedback on Buff

```text
SayLine "Feel the power of ancient magic!"
  SideEffects:
    + Apply Gameplay Effect
        EffectClass:       GE_AncientStrengthBuff
        bApplyToInstigator: true
    + Trigger Gameplay Cue
        CueTag:            GameplayCue.UI.BuffFlash
        bTriggerOnInstigator: true
        Mode:              Execute
```

---

## When Action Node, When SideEffect?

```text
Action Node:
[SayLine "Here is your payment."]
  │
  ▼
[Apply Gameplay Effect]    ← main step, deserves its own box
  EffectClass: GE_GoldReward
  │
  ▼
[Exit]

SideEffect:
[SayLine "Thank you!"]    ← the line is the main step
  SideEffects:
    + Add Tag: Story.NPC.ThankYouSaid     ← happens on the side
    + Trigger Cue: GameplayCue.Speech.Thanks
```

**Rule of thumb:** If the GAS effect is the reason this section of the graph exists → Action Node. If it happens as a companion action when entering another node → SideEffect.

> 📸 **Image placeholder:** `action-vs-sideeffect-graph.png` — Two graph excerpts side by side: left as Action Node in its own box, right as SideEffect pill on a SayLine.
> *Setup:* Two example graphs in the same asset placed side by side. Left side: `SayLine → ApplyEffect Node (blue, large) → Exit`. Right side: `SayLine` with expanded SideEffects array, containing `ApplyEffect` pill. Both sides labeled: "As Action Node" and "As SideEffect".

{% hint style="info" %}
**Want to build custom GAS actions?** Create a Blueprint subclass of `UMayDialogueSideEffect` and override `Execute Side Effect`. More in [Creating Custom GAS Nodes](extending.md).
{% endhint %}
