# Fire Event

Feuert einen `FGameplayTag`-Event. Das MayDialogue-Subsystem broadcastet `OnDialogueEventFired`; externe Systeme und Wait-Nodes können darauf reagieren.

## Runtime-Verhalten

`ExecuteNode` ruft `Instance::BroadcastDialogueEvent(EventTag)` und gibt Advance zurück.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `EventTag` | `FGameplayTag` | Das zu feuernde Event. |

## Typisches Pattern

Als Synchronisations-Punkt mit dem Rest des Spiels:

```
[SayLine: "Hörst du das? Es kommt von oben."]
  │
  ▼
[FireEvent: Story.Dialog.MonsterRevealed]
  │
  ▼
[SayLine: "Wir sollten verschwinden."]
```

Ein separates AI-System hört auf `Story.Dialog.MonsterRevealed` und triggert das Monster-Encounter.

## Anmerkungen

* Events sind **instant** – kein Return-Value, kein Handshake.
* Wait-Nodes mit `WaitEventTag=Story.Dialog.MonsterRevealed` würden durch dieses FireEvent aufwachen.
* Events landen auch im `OnDialogueEventFired`-Delegate des Subsystems – generische Listener können sich anhängen, ohne den Dialog-Code zu ändern.
