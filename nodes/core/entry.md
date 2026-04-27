# Entry

Der Entry-Node ist der Startpunkt jedes Dialog-Assets. Jedes Asset hat genau einen — er wird beim Erstellen automatisch platziert und kann nicht gelöscht werden.

## Wann setze ich ihn ein?

- Immer: er ist bereits da. Du musst ihn nicht manuell hinzufügen.
- Verbinde seinen Output-Pin mit dem ersten Node, der beim Dialogstart ausgeführt werden soll.
- Füge SideEffects hinzu, wenn beim Start sofort eine Aktion ausgelöst werden soll (z.B. eine Variable initialisieren oder einen Tag setzen).

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `EntryDescription` | `FText` | leer | Kommentar für den Designer — erscheint nicht im Spiel. |
| `SideEffects` | Array | leer | Inline-Aktionen, die beim Dialogstart ausgeführt werden (z.B. Variable setzen, Tag hinzufügen). |
| `FailBehavior` | Enum | `Skip` | Verhalten, wenn Node-Requirements fehlschlagen (geerbt von Base). |
| `EditorComment` | `FText` | leer | Graph-Notiz, kein Laufzeit-Effekt. |

{% hint style="info" %}
`EntryDescription` hilft Teams, den Einstiegspunkt zu verstehen — besonders wenn du mehrere Sub-Graphen mit eigenen Entry-Nodes im selben Asset hast.
{% endhint %}

> 📸 **Bild-Platzhalter:** `entry-node-graph.png` — Entry-Node im Graph mit Output-Verbindung.
> *Setup:* Asset `DA_Bsp_Entry` öffnen. Sichtbar: `Entry`-Node (grüne Titelleiste, kompakt) mit gefülltem `EntryDescription = "Erster Kontakt mit dem Wächter"`. Output-Pin verbunden mit einem `SayLine`-Node rechts davon. Im Body des Entry-Nodes eine SideEffect-Pill sichtbar: `AddTag: Story.DialogStarted`.

> 📸 **Bild-Platzhalter:** `entry-details-panel.png` — Details-Panel des Entry-Nodes.
> *Setup:* Entry-Node auswählen. Im Details-Panel sichtbar: `EntryDescription = "Erster Kontakt mit dem Wächter"`, Abschnitt `SideEffects` mit einem Eintrag `AddTag: Story.DialogStarted`.

## Mini-Beispiel

```text
[Entry]
  SideEffect: AddTag Story.Met.Guard
  │
  ▼
[SayLine: Wächter | "Halt! Wer bist du?"]
  │
  ▼
[PlayerChoice: ...]
```

> 📸 **Bild-Platzhalter:** `entry-example-graph.png` — Beispielgraph mit Entry + SideEffect.
> *Setup:* Graph mit drei Nodes: `Entry` (SideEffect-Pill `AddTag: Story.Met.Guard`) → `SayLine "Halt! Wer bist du?"` → `PlayerChoice`. Verbindungen: Entry-Output → SayLine-Input, SayLine-Output → PlayerChoice-Input.

## Häufige Fallstricke

- **Kein Output-Pin verbunden**: Der Compiler meldet einen Error. Verbinde den Entry-Output immer mit dem ersten inhaltlichen Node.
- **Mehrere Entry-Nodes**: Der Validator meldet einen Error — nur einer pro Hauptgraph erlaubt. (Sub-Graphen haben ihren eigenen Entry-Node, das ist korrekt.)
