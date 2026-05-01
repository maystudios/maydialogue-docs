---
description: Fire a GameplayTag event and trigger external systems from within the dialogue.
---

# Fire Event

Broadcasts an `FGameplayTag` as an event to all listeners. The dialogue continues immediately — no waiting, no handshake. Ideal for synchronizing game systems outside the dialogue.

## When to use

- **Trigger monster encounter** — NPC says "It's coming!", `Story.Dialog.MonsterRevealed` fires, AI system starts the encounter.
- **Trigger cinematic** — Dialogue reaches an important point, a separate Sequencer system listens to the event.
- **Notify quest system** — Player accepts a mission, `Quest.Accepted.FindKey` starts the quest logic.
- **Sound designer hook** — Audio Blueprint listens to `Dialog.Moment.Climax` and starts a music stinger.

---

> 📸 **Image placeholder:** `fire-event-node.png` — "Fire Event" Node in the MayDialogue graph.
> *Setup:* Node alone visible, title bar "Fire Event" (category color: yellow/Data). Subtitle shows: `EventTag = Story.Dialog.MonsterRevealed`. Input and Output pins visible.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `EventTag` | `FGameplayTag` | The GameplayTag broadcast as an event. |

---

> 📸 **Image placeholder:** `fire-event-details.png` — Details panel of the Node.
> *Setup:* Select the Node. In the Details panel: `EventTag = Quest.Accepted.FindKey`. Tag picker open with hierarchy preview.

---

## Action Node or SideEffect Sub-Node?

If firing the event is the **contentually significant moment** in the graph (the dialogue step represents the turning point), use the Action Node — it is clearly visible and debuggable. If the event is just a silent side effect of a SayLine (e.g. "every time this line plays"), attach it as a SideEffect pill.

---

## Example: Synchronizing a monster encounter

```text
[SayLine: Companion "Do you hear that? It's coming from above."]
  │
  ▼
[FireEvent: EventTag=Story.Dialog.MonsterRevealed]
  │
  ▼
[SayLine: Companion "We should get out of here!"]
  │
  ▼
[Exit]
```

An AI Blueprint subscribes to `OnDialogueEventFired` and reacts to `Story.Dialog.MonsterRevealed`.

> 📸 **Image placeholder:** `fire-event-example-graph.png` — Graph snippet of the example above.
> *Setup:* Four Nodes: SayLine (Companion) → FireEvent (Story.Dialog.MonsterRevealed in subtitle) → SayLine (Companion) → Exit. All pins connected.

---

## Pitfalls

{% hint style="info" %}
Events are **fire-and-forget** — no return value, no confirmation handshake. The dialogue does not wait for a listener to have reacted. If you want to wait for a result, use a `Wait` Node in combination with an external system that sends back a counter-event after processing.
{% endhint %}

- `OnDialogueEventFired` on the MayDialogue subsystem is the generic listener point — any external system can hook in without changing the dialogue code.
- `EventTag` should correspond to a defined tag from the project's tag tree. An unregistered tag throws no errors, but results in dead code.
- Wait Nodes with a matching `WaitEventTag` are awakened by this FireEvent — useful for dialogue-internal sync points.
