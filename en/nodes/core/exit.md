# Exit

The Exit Node ends the dialogue. An asset can have multiple Exit Nodes to model different outcomes — for example a positive and a negative conclusion.

## When should I use it?

- At the end of every dialogue — at least one Exit Node is required (the validator warns without one).
- Multiple exits when the dialogue should have different results that external systems (Quest, Achievement) evaluate differently.
- With SideEffects when a final action should be executed on completion (e.g. set a quest tag).
- As a merge point: multiple paths can lead into the same Exit Node.

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `ExitStatus` | `EMayDialogueExitStatus` | `Completed` | `Completed` = clean conclusion; `Failed` = premature or bad ending. Passed through in the `OnDialogueEnded` delegate. |
| `SideEffects` | Array | empty | Inline actions executed last, before the dialogue ends. |
| `FailBehavior` | Enum | `Skip` | Behavior when Node Requirements fail (inherited from Base). |
| `EditorComment` | `FText` | empty | Graph note, no runtime effect. |

> 📸 **Image placeholder:** `exit-node-graph.png` — Two Exit Nodes in the graph with different statuses.
> *Setup:* Graph snippet with `PlayerChoice` (two outputs). Output 0 → `Exit` with `ExitStatus = Completed` (red title bar), Output 1 → `Exit` with `ExitStatus = Failed` (darker red title bar). Both Exit Nodes labeled; the Completed exit has a SideEffect pill `AddTag: Quest.Guard.Passed`.

> 📸 **Image placeholder:** `exit-details-panel.png` — Details panel of an Exit Node.
> *Setup:* Select the Exit Node with `ExitStatus = Completed`. In the Details panel: `ExitStatus = Completed`, `SideEffects` array with one entry.

## Mini Example

```text
[PlayerChoice]
  ├─ Choice 0 "Friend"  ──► [SayLine: "You may pass."]   ──► [Exit: Completed]
  └─ Choice 1 "Enemy"   ──► [SayLine: "Get out!"]         ──► [Exit: Failed]
```

> 📸 **Image placeholder:** `exit-example-graph.png` — Complete mini-graph with two Exit Nodes.
> *Setup:* `Entry` → `SayLine "What is the password?"` → `PlayerChoice` with two Choices. Choice 0 leads to `SayLine "Welcome"` → `Exit Completed`. Choice 1 leads to `SayLine "Away with you"` → `Exit Failed`. All connections visible.

## Common Pitfalls

- **No Exit Node in the asset**: The validator warns — without an Exit, a dialogue can loop endlessly. An Exit Node is also required in sub-graphs.
- **ExitStatus set incorrectly**: External systems listening to `OnDialogueEnded` use the status for branching decisions. Forgetting a `Failed` status means quest systems will never report a failed conclusion.

{% hint style="info" %}
`OnDialogueEnded` delivers `(Asset, ExitStatus, Duration, Instigator, Target)`. Bind your quest system to this delegate to react to the status — without touching the dialogue code.
{% endhint %}
