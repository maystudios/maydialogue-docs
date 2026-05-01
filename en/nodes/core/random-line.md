# Random Line

The RandomLine Node randomly selects one of its output paths at runtime. Typical for greeting variants, idle comments, or atmospheric repetition without stale repetition.

## When should I use it?

- For NPC greetings that differ slightly each conversation.
- For ambient dialogue lines an NPC mutters randomly.
- When a certain result should appear more often than others (weights via `OutputWeights`).
- For test scenarios with reproducible results (`bDeterministicReplay`).

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `OutputWeights` | `TArray<float>` | empty | Weights per output pin. Must have the same count as output connections. Empty or wrong count = equal weights. |
| `bNoRepeat` | `bool` | `true` | Prevents the same selection twice in a row (per dialogue instance). |
| `bDeterministicReplay` | `bool` | `false` | When `true`, a stable random seed is used — the same instance always delivers the same sequence (useful for tests and replays). |
| `FixedSeed` | `int32` | `0` | Only active when `bDeterministicReplay = true`. `0` = derive seed from instance data; value ≠ 0 = this exact seed. |

{% hint style="info" %}
`bNoRepeat` only prevents repetitions **within a single dialogue instance**. A new instance starts without history — two consecutive conversations can draw the same line.
{% endhint %}

> 📸 **Image placeholder:** `randomline-node-graph.png` — RandomLine Node with three output connections.
> *Setup:* RandomLine Node with three output pins on the right, connected to three SayLine Nodes (`"You again."`, `"Back? Good for you."`, `"Another day, another you."`). In the Details panel `bNoRepeat = true`, `OutputWeights = (1.0, 1.0, 2.0)` (third line twice as likely).

> 📸 **Image placeholder:** `randomline-details-panel.png` — Details panel of the RandomLine Node.
> *Setup:* Select the RandomLine Node. In the Details panel: `OutputWeights = (1.0, 1.0, 2.0)`, `bNoRepeat = true`, `bDeterministicReplay = false`, `FixedSeed` greyed out.

## Mini Example

```text
[Entry]
  │
  ▼
[RandomLine]
  OutputWeights: [1.0, 1.0, 2.0]
  bNoRepeat: true
  ├─ Output 0 ──► [SayLine: Guard | "You again."]
  ├─ Output 1 ──► [SayLine: Guard | "Back? Good for you."]
  └─ Output 2 ──► [SayLine: Guard | "Another day, another you."]
                                    [all → PlayerChoice]
```

> 📸 **Image placeholder:** `randomline-example-graph.png` — Entry → RandomLine → three SayLine variants → shared PlayerChoice.
> *Setup:* `Entry` → `RandomLine` (bNoRepeat, Weights 1/1/2) → three `SayLine` Nodes each with a greeting variant. All three SayLine outputs lead into the same `PlayerChoice` Node. All connections visible.

## Common Pitfalls

- **`OutputWeights` wrong length**: If the number of weights does not match the output pins, all are weighted equally. The graph shows no warning — just count them.
- **RandomLine as SayLine replacement**: The RandomLine Node has no `SpeakerTag` or `DialogueText` of its own. The actual lines live in the connected SayLine Nodes. The RandomLine Node only selects the path.
