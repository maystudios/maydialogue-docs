---
description: Set a LooseGameplayTag on the player or an NPC — lightweight, no GE overhead.
---

# Add Tag

Adds an `FGameplayTag` directly via `AddLooseGameplayTag` to an `AbilitySystemComponent`. No GameplayEffect overhead — ideal for dialogue state flags that persist beyond the dialogue.

## When to use

- **Dialogue state flag on NPC** — Guard "knows a secret": `Story.Guard.KnowsSecret` on the guard's ASC.
- **Player has experienced something** — `Story.Player.HeardProphecy` ensures this dialogue branch is only accessible once.
- **Switch for other systems** — Quest system, AI StateTree, or other dialogues check this tag via `HasTag`.
- **Unlock mechanism** — NPC only trusts the player once `Story.Merchant.TrustGained` is set.

---

> 📸 **Image placeholder:** `add-tag-node.png` — "Add Tag" Node in the MayDialogue graph.
> *Setup:* Node alone, title bar "Add Tag" (category color: purple/GAS). Subtitle shows: `Story.Guard.KnowsSecret → Target`. Input and Output pins visible.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `TagToAdd` | `FGameplayTag` | The tag to add. |
| `bAddToInstigator` | `bool` | `true` = tag goes to the player's ASC. `false` = tag goes to the target NPC's ASC. Default: `true`. |

---

> 📸 **Image placeholder:** `add-tag-details.png` — Details panel with NPC target setup.
> *Setup:* Select the Node. In the Details panel: `TagToAdd = Story.Guard.KnowsSecret`, `bAddToInstigator = false`.

---

## Action Node or SideEffect Sub-Node?

If setting the tag is the **primary content of the step** (the graph represents the moment the NPC learns something), use the Action Node. If the tag is just silent background tracking of a SayLine (every time the player hears this line), attach it as a SideEffect pill to the SayLine.

---

## LooseTag vs. GameplayEffect Tag

| | LooseTag (this Node) | GameplayEffect Tag |
|---|---|---|
| Persistence | Until explicitly removed ([Remove Tag](remove-tag.md)) | Until GE ends or expires through stacking |
| Overhead | Very low | GE spec + modifier calculation |
| Replication | Limited (minimal mode) | Full via GAS replication |
| Typical use case | Dialogue flags, story state | Gameplay states with lifetime |

For story flags like "has greeted me", "knows my secret", LooseTag is the right choice.

---

## Example: Setting an NPC knowledge flag

```text
[PlayerChoice: "I'll tell you a secret..."]
  │
  ▼
[AddTag: TagToAdd=Story.Guard.KnowsSecret, bAddToInstigator=false]
  │
  ▼
[SayLine: NPC "No one must know..."]
  │
  ▼
[Exit]
```

> 📸 **Image placeholder:** `add-tag-example-graph.png` — Graph snippet of the example above.
> *Setup:* Four Nodes: PlayerChoice → AddTag (Story.Guard.KnowsSecret in subtitle, "→ Target") → SayLine (NPC) → Exit. All pins connected.

---

## Pitfalls

{% hint style="info" %}
`AddLooseGameplayTag` is **idempotent** — adding the same tag multiple times has no stack effect. The tag is either present or not. If you need a stack-based counter, use an Int attribute via `Apply Effect` instead.
{% endhint %}

- This action is part of the GAS integration and usable without additional setup. It accesses the Gameplay Ability System — make sure your characters have an `AbilitySystemComponent`.
- The target (Instigator or target NPC) must have a `UAbilitySystemComponent`, otherwise it is a no-op + log warning.
- LooseTags are not automatically replicated to other clients in Minimal replication mode — for multiplayer state, prefer `Apply Effect` with tag granting.
- Partner Node: [Remove Tag](remove-tag.md).
