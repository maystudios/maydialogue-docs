# Exit

Der Exit-Node beendet den Dialog. Ein Asset kann mehrere Exit-Nodes haben, um verschiedene Ausgänge zu modellieren — zum Beispiel einen positiven und einen negativen Abschluss.

## Wann setze ich ihn ein?

- Am Ende jedes Dialogs — mindestens ein Exit-Node ist Pflicht (ohne ihn warnt der Validator).
- Mehrere Exits, wenn der Dialog unterschiedliche Ergebnisse haben soll, die externe Systeme (Quest, Achievement) unterschiedlich auswerten.
- Mit SideEffects, wenn beim Abschluss noch eine letzte Aktion ausgeführt werden soll (z.B. Quest-Tag setzen).
- Als Zusammenführungspunkt: mehrere Pfade können in denselben Exit-Node münden.

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `ExitStatus` | `EMayDialogueExitStatus` | `Completed` | `Completed` = sauberer Abschluss; `Failed` = vorzeitiges oder schlechtes Ende. Wird im `OnDialogueEnded`-Delegate durchgereicht. |
| `SideEffects` | Array | leer | Inline-Aktionen, die als letztes vor dem Ende ausgeführt werden. |
| `FailBehavior` | Enum | `Skip` | Verhalten, wenn Node-Requirements fehlschlagen (geerbt von Base). |
| `EditorComment` | `FText` | leer | Graph-Notiz, kein Laufzeit-Effekt. |

> 📸 **Bild-Platzhalter:** `exit-node-graph.png` — Zwei Exit-Nodes im Graph mit unterschiedlichem Status.
> *Setup:* Graph-Ausschnitt mit `PlayerChoice` (zwei Outputs). Output 0 → `Exit` mit `ExitStatus = Completed` (rote Titelleiste), Output 1 → `Exit` mit `ExitStatus = Failed` (dunklere rote Titelleiste). Beide Exit-Nodes beschriftet, der Completed-Exit hat eine SideEffect-Pill `AddTag: Quest.Guard.Passed`.

> 📸 **Bild-Platzhalter:** `exit-details-panel.png` — Details-Panel eines Exit-Nodes.
> *Setup:* Exit-Node mit `ExitStatus = Completed` auswählen. Im Details-Panel sichtbar: `ExitStatus = Completed`, `SideEffects`-Array mit einem Eintrag.

## Mini-Beispiel

```text
[PlayerChoice]
  ├─ Choice 0 "Freund"  ──► [SayLine: "Passiere."]   ──► [Exit: Completed]
  └─ Choice 1 "Feind"   ──► [SayLine: "Verschwinde!"] ──► [Exit: Failed]
```

> 📸 **Bild-Platzhalter:** `exit-example-graph.png` — Vollständiger Mini-Graph mit zwei Exit-Nodes.
> *Setup:* `Entry` → `SayLine "Was ist das Passwort?"` → `PlayerChoice` mit zwei Choices. Choice 0 führt zu `SayLine "Willkommen"` → `Exit Completed`. Choice 1 führt zu `SayLine "Weg mit dir"` → `Exit Failed`. Alle Verbindungen sichtbar.

## Häufige Fallstricke

- **Kein Exit-Node im Asset**: Der Validator warnt — ohne Exit kann ein Dialog endlos loopen. In Sub-Graphen ist ein Exit-Node ebenfalls Pflicht.
- **ExitStatus falsch gesetzt**: Externe Systeme, die auf `OnDialogueEnded` lauschen, nutzen den Status für Fallunterscheidungen. Einen `Failed`-Status zu vergessen bedeutet, dass Quest-Systeme einen erfolgreichen Abschluss nie melden.

{% hint style="info" %}
`OnDialogueEnded` liefert `(Asset, ExitStatus, Duration, Instigator, Target)`. Binde dein Quest-System an diesen Delegate, um auf den Status zu reagieren — ohne den Dialog-Code anzufassen.
{% endhint %}
