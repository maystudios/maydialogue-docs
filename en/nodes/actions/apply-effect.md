---
description: Apply a UGameplayEffect to the player or a target NPC — directly from the dialogue.
---

# Apply Effect

Applies a `UGameplayEffect` to the Instigator (player) or a target Participant. Useful anywhere a dialogue should trigger a real gameplay consequence.

## When to use

- **NPC heals player** — Healer speaks a prayer, `GE_Heal` is applied to the Instigator.
- **Poison as punishment** — Player drinks a suspicious potion on NPC advice, `GE_Poison` hits the player.
- **Reputation system** — Player gives money to the merchant, `GE_ReputationUp` increases a reputation attribute on the NPC (`bApplyToInstigator=false`).
- **Buff after ritual** — Long dialogue conclusion grants the player a temporary strength buff.

---

> 📸 **Image placeholder:** `apply-effect-node.png` — "Apply Effect" Node in the MayDialogue graph.
> *Setup:* Node alone visible, title bar "Apply Effect" (category color: purple/GAS). Subtitle shows: `EffectClass = GE_Heal`, `bApplyToInstigator = true`. Input and Output pins visible.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `EffectClass` | `TSubclassOf<UGameplayEffect>` | The GameplayEffect to apply. |
| `EffectLevel` | `float` | Effect level for magnitude scaling. Minimum: 0. Default: `1.0`. |
| `bApplyToInstigator` | `bool` | `true` = player (self buff/debuff). `false` = target NPC. |

---

> 📸 **Image placeholder:** `apply-effect-details.png` — Details panel with filled values.
> *Setup:* Select the Node. In the Details panel: `EffectClass = GE_ReputationUp`, `EffectLevel = 1.0`, `bApplyToInstigator = false`.

---

## Action Node or SideEffect Sub-Node?

If applying the effect is the **main gameplay statement** of this step (e.g. "the player is healed now"), use the Action Node. If the effect just runs in the background as a side effect of a SayLine (e.g. slight stamina regeneration while listening), attach it as a SideEffect pill to the SayLine.

---

## Example: Healer heals the player

```text
[SayLine: Healer "Let me see your wounds."]
  │
  ▼
[ApplyEffect: EffectClass=GE_HealLarge, Level=1.0, bApplyToInstigator=true]
  │
  ▼
[SayLine: Healer "You are better now."]
```

> 📸 **Image placeholder:** `apply-effect-example-graph.png` — Graph snippet of the example above.
> *Setup:* Three Nodes: SayLine (Healer) → ApplyEffect (GE_HealLarge visible in subtitle) → SayLine (Healer). Pins connected.

---

## Pitfalls

{% hint style="warning" %}
The Node is a **silent no-op** if `EffectClass` is empty or no `AbilitySystemComponent` is found. You will see a log warning — but no editor error. Always verify that the Instigator and Target have an ASC.
{% endhint %}

{% hint style="info" %}
**The Instigator is always the Source of the effect** — even when `bApplyToInstigator=false`. This matters if your `UGameplayEffect` uses an `MMC_*` calculation that reads Source attributes (e.g. `Source.Strength`).
{% endhint %}

- This action is part of the GAS integration and usable without additional setup. It accesses the Gameplay Ability System — make sure your characters have an `AbilitySystemComponent`.
- GE level `0.0` is possible, but results in a zero-value effect with most standard magnitude formulas.
