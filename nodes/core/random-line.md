# Random Line

Der RandomLine-Node wählt zur Laufzeit einen seiner Output-Pfade zufällig aus. Typisch für Begrüßungsvarianten, Idle-Kommentare oder atmosphärische Wiederholungen ohne stumpfe Wiederholung.

## Wann setze ich ihn ein?

- Für NPC-Begrüßungen, die sich bei jedem Gespräch leicht unterscheiden.
- Für ambient-Dialog-Zeilen, die ein NPC zufällig murmelt.
- Wenn ein bestimmtes Ergebnis häufiger erscheinen soll als andere (Gewichte via `OutputWeights`).
- Für Test-Szenarien mit reproduzierbaren Ergebnissen (`bDeterministicReplay`).

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `OutputWeights` | `TArray<float>` | leer | Gewichte pro Output-Pin. Muss dieselbe Anzahl haben wie Output-Verbindungen. Leer oder falsche Anzahl = gleiche Gewichte. |
| `bNoRepeat` | `bool` | `true` | Verhindert dieselbe Auswahl zweimal hintereinander (pro Dialog-Instance). |
| `bDeterministicReplay` | `bool` | `false` | Wenn `true`, wird ein stabiler Zufalls-Seed genutzt — gleiche Instance liefert immer dieselbe Sequenz (nützlich für Tests und Replays). |
| `FixedSeed` | `int32` | `0` | Nur aktiv wenn `bDeterministicReplay = true`. `0` = Seed aus Instance-Daten ableiten; Wert ≠ 0 = dieser exakte Seed. |

{% hint style="info" %}
`bNoRepeat` verhindert nur Wiederholungen **innerhalb einer Dialog-Instance**. Eine neue Instance startet ohne Historie — zwei aufeinanderfolgende Gespräche können dieselbe Zeile ziehen.
{% endhint %}

> 📸 **Bild-Platzhalter:** `randomline-node-graph.png` — RandomLine-Node mit drei Output-Verbindungen.
> *Setup:* RandomLine-Node mit drei Output-Pins rechts, verbunden mit drei SayLine-Nodes (`"Du wieder."`, `"Zurück? Gut für dich."`, `"Noch ein Tag, noch du."`). Im Details-Panel `bNoRepeat = true`, `OutputWeights = (1.0, 1.0, 2.0)` (dritte Zeile doppelt so wahrscheinlich).

> 📸 **Bild-Platzhalter:** `randomline-details-panel.png` — Details-Panel des RandomLine-Nodes.
> *Setup:* RandomLine-Node auswählen. Im Details-Panel sichtbar: `OutputWeights = (1.0, 1.0, 2.0)`, `bNoRepeat = true`, `bDeterministicReplay = false`, `FixedSeed` ausgegraut.

## Mini-Beispiel

```text
[Entry]
  │
  ▼
[RandomLine]
  OutputWeights: [1.0, 1.0, 2.0]
  bNoRepeat: true
  ├─ Output 0 ──► [SayLine: Wächter | "Du wieder."]
  ├─ Output 1 ──► [SayLine: Wächter | "Zurück? Gut für dich."]
  └─ Output 2 ──► [SayLine: Wächter | "Noch ein Tag, noch du."]
                                    [alle → PlayerChoice]
```

> 📸 **Bild-Platzhalter:** `randomline-example-graph.png` — Entry → RandomLine → drei SayLine-Varianten → gemeinsamer PlayerChoice.
> *Setup:* `Entry` → `RandomLine` (bNoRepeat, Weights 1/1/2) → drei `SayLine`-Nodes mit je einer Begrüßungsvariante. Alle drei SayLine-Outputs führen in denselben `PlayerChoice`-Node. Verbindungen alle sichtbar.

## Häufige Fallstricke

- **`OutputWeights` falsche Länge**: Wenn die Anzahl der Gewichte nicht mit den Output-Pins übereinstimmt, werden alle gleich gewichtet. Der Graph zeigt keine Warnung — einfach nachzählen.
- **RandomLine als SayLine-Ersatz**: Der RandomLine-Node hat keinen eigenen `SpeakerTag` oder `DialogueText`. Die eigentlichen Zeilen leben in den verbundenen SayLine-Nodes. Der RandomLine-Node wählt nur den Pfad.
