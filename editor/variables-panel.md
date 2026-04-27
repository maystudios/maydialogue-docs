---
description: Variablen anlegen, Typen und Scope wählen, Default-Werte setzen und im Graph nutzen.
---

# Variables-Panel

Das Variables-Panel ist der Ort, an dem du **alle Variablen eines Dialog-Assets** deklarierst. Variablen erlauben dir, Zustände innerhalb eines Dialogs zu speichern und Verzweigungen davon abhängig zu machen.

> 📸 **Bild-Platzhalter:** `variables-panel-overview.png` — Variables-Panel mit vier Einträgen unterschiedlicher Typen und Scopes.
> *Setup:* Variables-Tab aktiv. Vier Zeilen: `HasAskedName` (Bool, Dialogue-Scope, Default: false), `AngerLevel` (Int, Dialogue-Scope, Default: 0), `Friendship` (Float, Participant-Scope, Default: 0.5), `SelectedTone` (Tag, Dialogue-Scope, leer). Alle Spalten sichtbar: Name, Typ, Scope, Default, Beschreibung.

## Variable anlegen

1. Variables-Tab öffnen.
2. **Add Variable** klicken.
3. **Name** eintippen (eindeutig im gewählten Scope).
4. **Typ** aus dem Dropdown wählen.
5. **Scope** festlegen: Dialogue oder Participant.
6. **Default-Wert** setzen.
7. Optional: **Beschreibung** für dich oder andere Designer eintragen.

> 📸 **Bild-Platzhalter:** `variables-panel-add-variable.png` — „Add Variable"-Button angeklickt, neue Zeile mit Cursor im Name-Feld.
> *Setup:* Variables-Panel mit zwei bestehenden Variablen. Dritte Zeile neu angelegt, Name-Feld aktiv, Typ-Dropdown geschlossen (Bool als Default). Roter Pfeil auf „Add Variable"-Button.

## Verfügbare Typen

| Typ | Wofür | Default-Beispiel |
| --- | --- | --- |
| **Bool** | Ja/Nein-Flags | `false` |
| **Int** | Zähler, Stufen | `0` |
| **Float** | Prozentwerte, Metriken | `0.0` |
| **String** | Freier Text (selten im Graph) | `""` |
| **Tag** | GameplayTag-Auswahl | *(leer)* |

## Scope wählen

**Dialogue-Scope** – Variable lebt nur für die Dauer dieses Dialog-Durchlaufs.

**Participant-Scope** – Variable ist an einen Participant gebunden und bleibt auch nach dem Dialog erhalten (solange der Participant im Speicher lebt).

### Entscheidungshilfe

```
Frage: Soll der NPC sich nach dem Dialog noch daran erinnern?
├─ Nein → Dialogue-Scope
└─ Ja   → Participant-Scope
```

**Typische Dialogue-Scope-Variablen:**
- `AngerLevel: Int` – Zählt Provokationen im Gespräch
- `HasAskedName: Bool` – Verhindert doppeltes Nachfragen
- `SelectedTone: Tag` – Merkt sich den gewählten Tonfall

**Typische Participant-Scope-Variablen:**
- `HasMet: Bool` – Erstkontakt gespeichert
- `Friendship: Float` – Beziehungsmetrik
- `KnownSecrets: Tag` – Welche Geheimnisse kennt der NPC

> 📸 **Bild-Platzhalter:** `variables-panel-scope-dropdown.png` — Scope-Dropdown einer Variable offen, „Dialogue" und „Participant" als Optionen.
> *Setup:* Variable `HasMet` selektiert. Scope-Feld als Dropdown aufgeklappt, zwei Optionen sichtbar. Roter Pfeil auf das Dropdown.

## Variablen im Graph nutzen

### Lesen (Requirement-Sub-Nodes)

Variablen-Werte liest du über **Requirement-Sub-Nodes** an Branch- oder Choice-Nodes:

- `CheckDialogueVariable` – prüft eine Dialogue-Scope-Variable
- `CheckParticipantVariable` – prüft eine Participant-Scope-Variable

Wähle den Sub-Node-Typ, trage den Variable-Namen ein, setze die Vergleichsbedingung.

### Schreiben (SetVariable)

Zwei Wege, um einen Wert zu setzen:

**Als Action-Node im Flow:**
Rechtsklick im Graph → Actions → Set Variable. Der Node erscheint als vollständige Box im Ablauf.

**Als SideEffect-Sub-Node:**
Rechtsklick auf einen SayLine- oder Choice-Node → Add SideEffect → Set Variable. Kompaktere Darstellung als Pill am Eltern-Node – passiert im Hintergrund, wenn der Eltern-Node ausgeführt wird.

```text
Beispiel-Graph: Anger-Tracking
[Entry] → [SayLine: "Du bist ein Narr!"]
          └─ SideEffect: SetVariable(AngerLevel = AngerLevel + 1)
        → [Branch: CheckDialogueVariable(AngerLevel >= 3)]
          ├─ True  → [SayLine: "Genug! Raus!"] → [Exit: Failed]
          └─ False → [SayLine: "Noch einmal..."] → [zurück]
```

> 📸 **Bild-Platzhalter:** `variables-setvariable-node.png` — SetVariable-Action-Node im Graph, Details-Panel zeigt Variable-Name und Wert.
> *Setup:* SetVariable-Node selektiert. Details zeigen: VariableName = „AngerLevel", VariableType = Int, NewValue = 1. Daneben im Graph sichtbar: Verbindung von SayLine zu diesem SetVariable-Node zu einem Branch-Node.

## Validator-Regel: Type Mismatch

Wenn zwei SetVariable-Nodes denselben Namen mit unterschiedlichen Typen beschreiben, meldet der Compiler einen **Variable Type Mismatch**-Error.

```
Variable „AngerLevel" deklariert als Int.
SetVariable-Node A: schreibt 1      (Int)  → OK
SetVariable-Node B: schreibt "High" (String) → ERROR
```

Deklariere die Variable immer im Variables-Panel mit dem korrekten Typ – dann vermeidest du diesen Fehler.

## Variablen im Preview und Debugger

Im **Preview-Runner** siehst du alle Dialogue-Scope-Variablen in Echtzeit und kannst ihre Werte manuell setzen, um Verzweigungen zu testen – ohne echte Spieler-Aktionen zu simulieren.

Im **PIE-Debugger** zeigt der Debugger-Watch-Tab alle aktiven Variablen der laufenden Instance: Dialogue-Scope-Werte direkt, Participant-Scope-Werte von den angebundenen Actors.

Siehe [Preview-Runner](preview-runner.md) und [Debugger](debugger.md) für Details.

> 📸 **Bild-Platzhalter:** `variables-preview-watch.png` — Preview-Runner-State-Bereich mit zwei Variables live angezeigt.
> *Setup:* Preview läuft, Dialog hat AngerLevel=1 erreicht. Unten im Preview-Panel: Variable-Liste zeigt `HasAskedName: false`, `AngerLevel: 1`. Roter Pfeil auf die Variable-Liste im Preview-State-Bereich.

Weiter: [Outline →](outline.md)
