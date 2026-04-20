# Wait

Der **Wait-Node** pausiert den Dialog, bis eine Bedingung eintritt: Zeit abgelaufen oder GameplayTag-Event gefeuert.

## Runtime-Verhalten

`ExecuteNode`:

1. Registriert einen Timer (wenn `WaitDuration > 0`) und/oder einen Event-Listener (wenn `WaitEventTag` valide).
2. Registriert sich als aktiver Async-Node.
3. Gibt kein direktes Advance – wartet auf Trigger.

Beim Trigger:

1. Entsprechende Callback-Methode.
2. `Instance::ForceTransitionToNode(NextNodeGuid)`.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `WaitDuration` | `float` | Warte so viele Sekunden. 0 = nicht. |
| `WaitEventTag` | `FGameplayTag` | Warte auf dieses GameplayEvent. |
| `WaitCondition` | `UMayDialogueRequirement*` | Pollt alle `ConditionCheckInterval` s. |
| `ConditionCheckInterval` | `float` | Poll-Intervall für WaitCondition. |
| `bRequireBoth` | `bool` | AND vs. OR für Duration + Event. |

## Pins

* **Input**.
* **Output** – fort, sobald Trigger greift.

## Modi

* **Nur Duration** → simple Pause.
* **Nur Event** → wartet auf externes Signal (z.B. Spieler-Aktion).
* **Beide, OR** → was zuerst eintritt, löst aus.
* **Beide, AND** → beides muss eingetreten sein (sobald Duration abgelaufen *und* Event gefeuert wurde).

## Typisches Pattern

Dramatische Pause:

```
[SayLine: "Ich... ich habe etwas zu beichten."]
  │
  ▼
[Wait: 1.5s]
  │
  ▼
[SayLine: "Es war alles meine Schuld."]
```

Warten auf externe Aktion:

```
[SayLine: "Folge mir zum Fenster."]
  │
  ▼
[Wait: EventTag="Story.PlayerAtWindow"]
  │
  ▼
[SayLine: "Schau nach draußen..."]
```

## Anmerkungen

* **Condition-Modus (Polling-Requirement)** ist im Code als Feature vorbereitet, aber noch nicht vollständig im Asset-Editor freigegeben (Backlog-Item 7).
* Bei **Abort** des Dialogs während eines Wait-Nodes wird der Timer im Cleanup getrennt. Falls nicht – der Callback prüft auf Weak-Pointer.
