# Wait

The Wait Node pauses the dialogue until one or more conditions occur: a timer runs out, a GameplayEvent is fired, or a condition becomes true. All three modes can be combined.

## When should I use it?

- For dramatic pauses between two sentences.
- When the dialogue should wait for a player action before continuing (e.g. "Go to the window").
- For cutscene timing: insert a fixed pause after an animation or sound.
- As a polling loop: wait until an attribute reaches a threshold.
- With AND semantics: only continue when time has elapsed **and** the event has fired.

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `WaitDuration` | `float` | `1.0` | Wait this many seconds. `0` = timer disabled. |
| `WaitEventTag` | `FGameplayTag` | empty | Wait for this GameplayEvent. Empty = event-wait mode disabled. |
| `WaitCondition` | `UMayDialogueRequirement*` (Instanced) | empty | Polls every `ConditionCheckInterval` seconds. Empty = condition mode disabled. |
| `ConditionCheckInterval` | `float` | `0.2` | Poll interval in seconds. Only active when `WaitCondition` is set. |
| `bRequireBoth` | `bool` | `false` | `false` = OR: any active condition is sufficient; `true` = AND: all active conditions must be met simultaneously. |

{% hint style="info" %}
A wait condition is considered "active" when it is configured: `WaitDuration > 0`, `WaitEventTag` valid, or `WaitCondition` set. When no condition is active, the Node passes through immediately.
{% endhint %}

> 📸 **Image placeholder:** `wait-node-graph.png` — Wait Node in the graph with Duration and EventTag.
> *Setup:* Wait Node with `WaitDuration = 2.0` and `WaitEventTag = Story.PlayerAtWindow`. Input pin connected to `SayLine "Follow me to the window."`. Output pin connected to `SayLine "Look outside..."`. Visible in the Wait Node body: Duration and Event pills.

> 📸 **Image placeholder:** `wait-details-panel.png` — Details panel of the Wait Node.
> *Setup:* Select the Wait Node. In the Details panel: `WaitDuration = 2.0`, `WaitEventTag = Story.PlayerAtWindow`, `WaitCondition` (empty), `ConditionCheckInterval = 0.2`, `bRequireBoth = false`.

## Modes

| Mode | Configuration | Behavior |
| --- | --- | --- |
| Duration only | `WaitDuration > 0` | Pauses for the specified time. |
| Event only | `WaitEventTag` valid | Waits for an external GameplayEvent. |
| Condition only | `WaitCondition` set | Polls the condition on the interval. |
| OR (default) | multiple active, `bRequireBoth = false` | Continues when any condition is met. |
| AND | multiple active, `bRequireBoth = true` | Continues when all active conditions are met. |

## Mini Example

**Dramatic pause:**

```text
[SayLine: NPC | "I... I have something to confess."]
  │
  ▼
[Wait: WaitDuration=1.5]
  │
  ▼
[SayLine: NPC | "It was all my fault."]
```

**Wait for external action:**

```text
[SayLine: NPC | "Follow me to the window."]
  │
  ▼
[Wait: WaitEventTag=Story.PlayerAtWindow]
  │
  ▼
[SayLine: NPC | "Do you see the camp out there?"]
```

> 📸 **Image placeholder:** `wait-example-graph.png` — Two demo graphs side by side: timer mode and event mode.
> *Setup:* Left: `SayLine "I have something to confess."` → `Wait (Duration=1.5)` → `SayLine "It was all my fault."`. Right: `SayLine "Follow me."` → `Wait (EventTag=Story.PlayerAtWindow)` → `SayLine "Do you see the camp?"`. Both graphs complete with connections.

## Common Pitfalls

- **`WaitDuration = 0` with empty Event and empty Condition**: The Node passes through immediately — this is correct, but a Wait without an active condition is redundant.
- **Aborting the dialogue during Wait**: Timers and event listeners are automatically cleaned up on abort/cleanup (via the async-state mechanism). Manual cleanup is not needed.
