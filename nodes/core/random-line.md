# Random Line

Der **RandomLine-Node** wählt zur Laufzeit **eine** seiner konfigurierten Text-Varianten aus und präsentiert sie wie eine SayLine.

## Runtime-Verhalten

`ExecuteNode`:

1. Zufällige Line aus `Lines[]` auswählen (gewichtet, wenn konfiguriert).
2. Wenn `bRememberSelection`: letzte Wahl ausschließen.
3. Message bauen, wie SayLine.
4. `ReceiveMessage`, Instance wartet auf Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `SpeakerTag` | `FGameplayTag` | Wie bei SayLine. |
| `Lines` | `TArray<FText>` | Text-Varianten. |
| `EmotionTags` | `FGameplayTagContainer` | Pro-Node (nicht pro-Line). |
| `bRememberSelection` | `bool` | Keine Wiederholung hintereinander. |
| `RandomSeed` | `int32` | 0 = echter Zufall; sonst deterministisch. |
| `AdvanceModeOverride` | `EMayDialogueAdvanceMode` | Wie bei SayLine. |

## Pins

* **Input**.
* **Output** – einer, unabhängig davon welche Line gewählt wurde.

## Typisches Pattern

Für variierte Begrüßungen:

```
[Entry]
  │
  ▼
[RandomLine: Wächter]
  Lines:
    - "Du wieder."
    - "Zurück? Gut für dich."
    - "Noch ein Tag, noch du."
  bRememberSelection: true
  │
  ▼
[PlayerChoice: ...]
```

## State per Instance

Die letzte Wahl wird **per Instance** getrackt – nicht auf dem Asset-Node. Das heißt: jede neue Dialog-Instance startet mit leerer Historie. Zwei aufeinanderfolgende Gespräche mit demselben NPC können dieselbe RandomLine wählen, sofern `bRememberSelection` nicht über den Dialog-Abschluss hinaus greifen soll (was es auch nicht tut).

## Anmerkungen

* Gewichte pro Line sind **in Vorbereitung** – aktuell sind alle Lines gleich gewichtet.
* Wenn nur eine Line konfiguriert ist, verhält sich der Node wie eine normale SayLine.
* Voice-Assets sind **nicht pro-Line** auswählbar – das heißt: RandomLines haben entweder gar kein Voice (Babel) oder dasselbe Voice-Asset für alle Varianten. Empfehlung: RandomLines nur für Text-Only-Stellen nutzen.
