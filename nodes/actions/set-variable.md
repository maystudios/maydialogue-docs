# Set Variable

Setzt eine Dialog- oder Participant-Variable.

## Runtime-Verhalten

`ExecuteNode`:

1. Löst die Ziel-Variable je nach `VariableScope` auf (Instance-DialogueVariables oder Participant-PersistentMemory).
2. Schreibt den typisierten Wert.
3. Broadcastet `OnVariableChanged`.
4. Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `VariableName` | `FName` | Name der Variable. |
| `VariableType` | `EMayDialogueVariableType` | Bool / Int / Float / String / Tag. |
| `VariableScope` | `EMayDialogueVariableScope` | Dialogue oder Participant. |
| `TargetParticipantTag` | `FGameplayTag` | Nur bei Participant-Scope: welcher Participant. |
| `BoolValue` / `IntValue` / `FloatValue` / `StringValue` / `TagValue` | typisiert | Der neue Wert. |

## Typisches Pattern

Vor einer Verzweigung ein Flag setzen:

```
[SayLine: "Ich verrate dir ein Geheimnis..."]
  │
  ▼
[SetVariable: HasHeardSecret=true, Scope=Participant]
  │
  ▼
[SayLine: "Der Schlüssel liegt im Turm."]
```

## Action-Node vs. SideEffect

Wenn das Setzen *der Hauptpunkt* dieses Graph-Schritts ist → Action-Node (eigene Box). Wenn es nebenbei beim Betreten einer SayLine passiert → SideEffect-Sub-Node an der SayLine.

## Anmerkungen

* **Participant-Scope-Schreiben ist als Backlog-Item 8 markiert.** Aktuell schreibt der Node nur in Dialogue-Scope. Workaround: Blueprint-SideEffect verwenden, der `Part->SetPersistentBool(...)` aufruft.
* **Tag-Wert im SetVariable** fehlt in der UI-Pfad-Implementierung (Backlog-Item 9).
* Der Name muss im Variables-Panel deklariert sein; sonst loggt der Node eine Warning.
