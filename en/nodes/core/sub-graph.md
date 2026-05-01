# SubGraph

The SubGraph Node expands an embedded sub-graph within the same asset. It works like [Link](link.md), but the target lives in the same asset — no separate asset needed.

## When should I use it?

- When a dialogue fragment only appears in this asset and is not shared with other assets.
- To keep complex graphs manageable: build repeated sequences (farewell, combat reaction) once and call them multiple times.
- For nested sub-dialogues (sub-graph within sub-graph) without needing to create separate assets.
- When you prefer breadcrumb navigation and step-into debugging in the editor (works better for SubGraphs than for cross-asset Links).

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `SubGraphName` | `FText` | empty | Display name of the sub-graph in the Node body, e.g. `"Combat Reaction"`. |
| `bReturnAfterSubGraph` | `bool` | `true` | `true` = return to the next Node after the sub-graph exits; `false` = sub-graph is terminal, no return. |

{% hint style="info" %}
**Opening:** Double-clicking the SubGraph Node opens the embedded graph as its own tab with a breadcrumb path. Go back with the breadcrumb or the tab list.
{% endhint %}

> 📸 **Image placeholder:** `subgraph-node-graph.png` — SubGraph Node in the main graph.
> *Setup:* Main graph with two SubGraph Nodes: `"Painful Farewell"` (input from `SayLine "You betrayed me!"`) and `"Happy Farewell"` (input from `SayLine "You are a friend."`). Both SubGraph Nodes show their name in the body. Output pins connected to `Exit`.

> 📸 **Image placeholder:** `subgraph-inner-graph.png` — Opened sub-graph "Painful Farewell" in its own tab.
> *Setup:* Tab `DA_Farewell > Painful Farewell` open. Visible: `Entry` → `SayLine "That was a mistake."` → `CameraShake` → `Exit`. Breadcrumb at top shows `DA_Farewell / Painful Farewell`.

> 📸 **Image placeholder:** `subgraph-details-panel.png` — Details panel of the SubGraph Node.
> *Setup:* Select the SubGraph Node "Painful Farewell". In the Details panel: `SubGraphName = "Painful Farewell"`, `bReturnAfterSubGraph = true`.

## Mini Example

```text
Main Graph:
  [SayLine: NPC | "You betrayed me!"]  ──► [SubGraph "Painful Farewell"] ──► [Exit: Failed]
  [SayLine: NPC | "You are a friend."] ──► [SubGraph "Happy Farewell"]   ──► [Exit: Completed]

SubGraph "Painful Farewell":
  [Entry] → [SayLine: NPC | "I will never forget this."] → [CameraShake] → [Exit]

SubGraph "Happy Farewell":
  [Entry] → [SayLine: NPC | "Until next time."] → [PlayAnimation: HugMontage] → [Exit]
```

> 📸 **Image placeholder:** `subgraph-example-graph.png` — Main graph with two SubGraph Nodes plus opened sub-graph in the background.
> *Setup:* Main tab shows two paths each with a SubGraph Node. Second tab `Painful Farewell` open with inner graph. Breadcrumb visible.

## Common Pitfalls

- **`bReturnAfterSubGraph = false` with Output pin connected**: This can confuse the compiler. If no return is desired, leave the Output pin of the SubGraph Node unconnected.
- **Sub-graph without Exit Node**: The sub-graph hangs after the last Node. Always add an Exit in the sub-graph.

## SubGraph vs. Link

| | SubGraph | Link |
| --- | --- | --- |
| Target | In the same asset | Another asset |
| Editor navigation | Tab with breadcrumb | Switch asset |
| Reuse | Only in this asset | Across multiple assets |
| Debugger step-into | Full | Rudimentary |

**Rule of thumb:** Fragment only in this asset? SubGraph. Should it be shared across multiple assets? Link.
