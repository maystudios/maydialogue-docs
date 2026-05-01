---
description: Remove a LooseGameplayTag from the player or an NPC.
---

# Remove Tag

Removes an `FGameplayTag` via `RemoveLooseGameplayTag` from an `AbilitySystemComponent`. Only affects tags that were set via `AddLooseGameplayTag` — GameplayEffect-granted tags are unaffected.

## When to use

- **Regain trust** — NPC forgives the player: remove `Story.Guard.Distrusts`.
- **Complete a quest phase** — After a goal is fulfilled, remove the tracking flag.
- **Reset after a choice** — Player switches sides in a story branch; delete the old faction flag.
- **State cleanup** — Dialogue end cleans up temporary flags that were only valid during the conversation.

---

> 📸 **Image placeholder:** `remove-tag-node.png` — "Remove Tag" Node in the MayDialogue graph.
> *Setup:* Node alone, title bar "Remove Tag" (category color: purple/GAS). Subtitle shows: `Story.Guard.Distrusts ← Target`. Input and Output pins visible.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `TagToRemove` | `FGameplayTag` | The tag to remove. |
| `bRemoveFromInstigator` | `bool` | `true` = tag is removed from the player's ASC. `false` = tag is removed from the target NPC's ASC. Default: `true`. |

---

> 📸 **Image placeholder:** `remove-tag-details.png` — Details panel with NPC target setup.
> *Setup:* Select the Node. In the Details panel: `TagToRemove = Story.Guard.Distrusts`, `bRemoveFromInstigator = false`.

---

## Action Node or SideEffect Sub-Node?

If removing the tag is the **semantic main point** of the step (the moment trust is regained), use the Action Node. If it is just silent cleanup at the end of a SayLine, attach it as a SideEffect pill to the Exit Node.

---

## Example: Regaining trust

```text
[SayLine: Guard "You have proven yourself. I trust you again."]
  │
  ▼
[RemoveTag: TagToRemove=Story.Guard.Distrusts, bRemoveFromInstigator=false]
  │
  ▼
[Exit Completed]
```

> 📸 **Image placeholder:** `remove-tag-example-graph.png` — Graph snippet of the example above.
> *Setup:* Three Nodes: SayLine (Guard) → RemoveTag (Story.Guard.Distrusts, "← Target" in subtitle) → Exit. All pins connected.

---

## Pitfalls

{% hint style="info" %}
`RemoveLooseGameplayTag` is **idempotent** — if the tag was not set, nothing happens (no error, no log). Safe to use without a prior HasTag check.
{% endhint %}

{% hint style="warning" %}
This Node removes **only LooseTags**. Tags granted by a `UGameplayEffect` remain. To remove GE tags, the effect itself must be removed (e.g. via `Apply Effect` with a `GE_Remove` class or through expiry).
{% endhint %}

- This action is part of the GAS integration and usable without additional setup. It accesses the Gameplay Ability System — make sure your characters have an `AbilitySystemComponent`.
- The target must have a `UAbilitySystemComponent` — otherwise no-op + log warning.
- Partner Node: [Add Tag](add-tag.md).
