---
description: Reuse the same dialogue block across multiple assets – Link node vs. SubGraph, decision guide and setup.
---

# Reusable Dialogue Fragments

## Scenario

Every merchant in the game ends their conversation with the same farewell block: *"Safe travels, wanderer."* followed by a random warning. Instead of copying this block into every asset, you extract it once. With five merchants you save five times the maintenance effort — and bug fixes apply in one place.

## What You Will Learn

- Point a Link node to an external fragment asset.
- Use `bReturnAfterExit` so the host dialogue continues after the fragment.
- SubGraph as an alternative for internal organization.
- When to use Link, when to use SubGraph.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.

## Mini-Graph

**Fragment asset `DA_Common_Farewell`:**

```text
[Entry]
   │
   ▼
[SayLine: "Safe travels, wanderer."]
   │
   ▼
[RandomLine: 3 warnings about the forests]
   │
   ▼
[Exit: Completed]
```

**Merchant asset `DA_Merchant_*`:**

```text
[... Merchant-specific conversation ...]
   │
   ▼
[Link → DA_Common_Farewell  bReturnAfterExit: true]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `linking-dialogues-graph-overview.png` — Two assets side by side: fragment asset on the left, merchant asset on the right with a Link node.
> *Setup:* Content Browser and two editor windows side by side. Left: `DA_Common_Farewell` with Entry → SayLine → RandomLine → Exit. Right: `DA_Merchant_A` with conversation block → Link node (portal icon) → Exit. Link node has `LinkedAsset = DA_Common_Farewell` visible in the Details panel.

## Step by Step – Link Variant

### 1. Create the Fragment Asset

New asset `DA_Common_Farewell`. Speaker: `Dialogue.Speaker.CurrentSpeaker` (generic token — see Pro Tip).

Graph: Entry → SayLine *"Safe travels, wanderer."* → RandomLine with three warnings → Exit (`Completed`).

### 2. Add the Link Node to the Merchant Dialogue

In the merchant asset, at the end of the actual conversation: from the last output pin → **Create Node → Link**.

### 3. Configure the Link Node

| Property | Value |
|----------|------|
| `LinkedAsset` | `DA_Common_Farewell` |
| `EntryGuid` | *(empty = default Entry)* |
| `bReturnAfterExit` | `true` |

### 4. Wire Up the Return

The Link node has an output pin that fires when the fragment ends. Connect it to your final Exit node.

> 📸 **Image placeholder:** `linking-dialogues-link-node-details.png` — Details panel of the Link node.
> *Setup:* Link node selected. Details shows: `LinkedAsset = DA_Common_Farewell`, `EntryGuid = (empty)`, `bReturnAfterExit = true (checkbox enabled)`.

{% hint style="info" %}
**Pro tip — Speaker tokens:** Instead of hard-coded speaker tags in the fragment, use `Dialogue.Speaker.CurrentSpeaker`. In the host asset, this token is mapped to the currently speaking merchant — so the fragment works for any merchant without modification.
{% endhint %}

## SubGraph Variant (for purely internal organization)

If the fragment is **not** needed in other assets but just to clean up your own graph:

From the last output pin → **Create Node → SubGraph**. Name it *"Farewell Block"*. Double-clicking opens the sub-graph in a new tab with breadcrumb navigation.

```text
[... Merchant conversation ...]
   │
   ▼
[SubGraph: "Farewell Block"]
   │ (internally: Entry → SayLine → RandomLine → Exit)
   ▼
[Exit: Completed]
```

## Link vs. SubGraph

| Criterion | Link | SubGraph |
|-----------|------|----------|
| Reuse across asset boundaries | Yes | No |
| Collapsible within own asset | No | Yes |
| Own Speakers scope | Yes | No (inherits from parent) |
| Bug fix applies everywhere | Yes (centrally) | No (locally) |
| Merge conflicts | Fewer (separate assets) | More (everything in one file) |

**Rule of thumb:** Reuse across multiple assets → **Link**. Only internal cleanup → **SubGraph**.

## Blueprint Triggering

Nothing changes for the caller. Standard start call — the fragment runs in the same Instance:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Merchant_A
   └─ ...
```

> 📸 **Image placeholder:** `linking-dialogues-bp-trigger.png` — Blueprint trigger on the merchant.
> *Setup:* Merchant actor Blueprint. `Event OnInteract` → `Start Dialogue (MayDialogueLibrary)`, `Asset = DA_Merchant_A`.

## Variations / Going Further

- Set a variable before the Link node (`SetVariable`) that the fragment reads — passing parameters without new assets. See [GAS-Driven Dialogue](gas-driven-dialogue.md).
- Vary the fragment based on player tags: Branch inside the fragment asset. All users benefit automatically.
- Nest multiple fragments: Link A → Link B → Link C. The scope stack manages the return.

## Troubleshooting

**Dialogue ends after the fragment, even though it should continue.**
`bReturnAfterExit = false` on the Link node. Or the Exit inside the fragment has `ExitStatus = Failed` and your host dialogue reacts to that with a different outcome. Check the flag.

**Infinite loop between host and fragment.**
The fragment links back to the host. The scope stack grows deeper than the limit (16). Use Branches instead of mutual Links.

**Speaker in fragment not resolved.**
The fragment asset doesn't know the speaker tag. Also add the speaker there, or use the token approach from the Pro Tip.
