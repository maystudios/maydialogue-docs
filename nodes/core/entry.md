# Entry

Der **Entry-Node** markiert den Startpunkt jedes Dialogs. **Genau einer pro Asset**, nicht löschbar.

## Runtime-Verhalten

Bei `Instance::StartDialogue(...)` wird der Entry-Node als erster besucht. Er hat keine eigene Logik – sein einziger Zweck ist, einen klaren Einstiegspunkt in den Graph zu markieren.

`ExecuteNode` ruft `SideEffects` auf (falls vorhanden) und liefert `Advance(FirstOutputGuid)`.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `EntryDescription` | `FText` | Kommentar für Designer (optional). Nicht UI-sichtbar. |
| `SideEffects` | Array | SideEffects, die beim Dialog-Start ausgeführt werden. Ideal für Initial-Setup. |

## Pins

* **Output** – zum nächsten Node im Graph. Wenn unverbunden: Compiler-Error.

## Typisches Pattern

```
Entry (SideEffect: SetVariable "HasMet" = true)
  │
  ▼
[SayLine: "Schön dich zu sehen."]
```

## Anmerkungen

* Der Entry-Node erscheint nicht im Details-Picker, weil er automatisch beim Erzeugen eines neuen Assets platziert wird.
* **Nicht löschbar** – der Schema verhindert Delete.
* Mehrere Entry-Nodes werden vom Validator als Error gemeldet.
