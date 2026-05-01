---
description: Eine Dialog-Variable setzen — Bool, Int, Float, String oder Tag.
---

# Set Variable

Schreibt einen Wert in eine benannte Dialog-Variable. Die Variable kann im Dialogue-Scope (nur diese Instanz) oder im Participant-Scope (persistentes Gedächtnis des NPCs) gespeichert werden.

## Wann nutzen

- **Geheimnis-Flag setzen** — Spieler hört etwas Wichtiges: `HasHeardSecret = true` für spätere Branch-Entscheidungen.
- **Quest-Fortschritt zählen** — Spieler liefert Items ab: `QuestItemCount` um 1 erhöhen (Int-Variable).
- **Sprecher-Zustand merken** — Welche Antwort hat der Spieler zuletzt gegeben? (`LastChoice = "Option_A"`).
- **Tag-Variable für komplexe Bedingungen** — Setze einen GameplayTag-Wert, den ein späterer Branch-Node auswertet.

---

> 📸 **Bild-Platzhalter:** `set-variable-node.png` — Node "Set Variable" im MayDialogue-Graphen.
> *Setup:* Node allein sichtbar, Title-Bar "Set Variable" (Kategorie-Farbe: gelb/Daten). Subtitle zeigt: `HasHeardSecret = true (Bool)`. Input- und Output-Pin sichtbar.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `VariableName` | `FName` | Name der Variable. Muss im Variables-Panel des Assets deklariert sein. |
| `VariableType` | `EMayDialogueVariableType` | `Bool` / `Int` / `Float` / `String` / `Tag` — bestimmt welches Wert-Feld aktiv ist. |
| `Scope` | `EMayDialogueVariableScope` | `Dialogue` = nur diese Instanz. `Participant` = persistentes Gedächtnis des NPCs. |
| `TargetParticipantTag` | `FGameplayTag` | Welcher Participant bekommt den Wert. Nur aktiv bei `Scope = Participant`. |
| `BoolValue` | `bool` | Wert wenn `VariableType = Bool`. |
| `IntValue` | `int32` | Wert wenn `VariableType = Int`. |
| `FloatValue` | `float` | Wert wenn `VariableType = Float`. |
| `StringValue` | `FString` | Wert wenn `VariableType = String`. |
| `TagValue` | `FGameplayTag` | Wert wenn `VariableType = Tag`. |

> Die Details-Panel zeigt immer nur das zum `VariableType` passende Wert-Feld an (EditConditionHides).

---

> 📸 **Bild-Platzhalter:** `set-variable-details.png` — Details-Panel mit Bool-Setup.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `VariableName = HasHeardSecret`, `VariableType = Bool`, `Scope = Dialogue`, `TargetParticipantTag = (leer, ausgegraut)`, `BoolValue = true`. Alle anderen Wert-Felder ausgeblendet.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn das Setzen der Variable der **inhaltliche Hauptpunkt** dieses Schrittes ist (der Graph-Schritt repräsentiert den Moment des Merkens), nimm den Action-Node. Wenn die Variable nur nebenbei beim Eintreten einer SayLine gesetzt wird (automatisches Tracking), hänge einen SideEffect-Sub-Node an die SayLine.

---

## Beispiel: Geheimnis-Flag setzen

```text
[SayLine: Alter Mann "Hör mir gut zu..."]
  │
  ▼
[SetVariable: HasHeardSecret=true, Scope=Dialogue]
  │
  ▼
[SayLine: Alter Mann "Der Schlüssel liegt im Turm."]
  │
  ▼
[Branch: Condition=HasHeardSecret → True → [Exit Completed]]
```

> 📸 **Bild-Platzhalter:** `set-variable-example-graph.png` — Graphausschnitt des obigen Beispiels.
> *Setup:* Vier Nodes: SayLine → SetVariable (HasHeardSecret=true im Subtitle) → SayLine → Branch. Alle Pins verbunden. Branch-Node zeigt True/False-Ausgangspins.

---

## Fallstricke

{% hint style="warning" %}
`VariableName` muss im **Variables-Panel** des Dialogue-Assets deklariert sein. Ein unbekannter Name führt zu einer Log-Warning und die Variable wird nicht geschrieben — kein harter Fehler, aber lautlos korrumpierte Logik.
{% endhint %}

{% hint style="info" %}
**Participant-Scope** ist für persistentes NPC-Gedächtnis gedacht (NPC erinnert sich nach Dialogue-Ende). Das Schreiben in diesen Scope funktioniert nur wenn der Participant eine entsprechende PersistentMemory-Komponente hat.
{% endhint %}

- Beim Wechsel des `VariableType` werden die anderen Wert-Felder ausgeblendet — der jeweilige Wert bleibt im Asset gespeichert, wird aber nicht genutzt.
- `OnVariableChanged` wird nach dem Schreiben gebroadcastet — externe Systeme können darauf reagieren.
