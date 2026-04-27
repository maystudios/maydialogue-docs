---
description: Dialog direkt im Editor testen – ohne PIE, mit GAS-State-Simulation, Culture-Wechsel und Live-Variablen.
---

# Preview-Runner

Der Preview-Runner spielt einen Dialog **direkt im Asset-Editor ab** – kein Level laden, kein PIE starten, kein Spieler spawnen. Du drückst Play, der Dialog läuft.

Das ist das Werkzeug, mit dem die meiste Dialog-Arbeit passiert. Eine Zeile ändern, sofort testen, wieder ändern – die Iterationszeit liegt unter einer Sekunde.

> 📸 **Bild-Platzhalter:** `preview-runner-active.png` — Preview-Panel mit aktivem Dialog. Sprecher-Portrait oben, Text mit Typewriter-Effekt, zwei Choice-Buttons sichtbar.
> *Setup:* Asset `DA_Guard_Greeting` im Preview-Runner. Sprecher „Wächter" mit Portrait oben links, Dialogtext „Halt! Wer bist du?" (halb eingetippt, Typewriter-Effekt). Darunter zwei Choice-Buttons: „Ein Freund des Königs." und „Das geht dich nichts an." Unten: Event-Log mit „Entry → SayLine_Guard_01".

## Starten

1. Toolbar-Button **Play Dialog** klicken – oder den Play-Button direkt im Preview-Tab.
2. Der Runner startet am Entry-Node.
3. Dialog läuft.

> 📸 **Bild-Platzhalter:** `preview-runner-start.png` — Toolbar mit rotem Pfeil auf Play-Dialog-Button, Preview-Tab daneben leer (vor dem Start).
> *Setup:* Asset offen, noch kein Preview aktiv. Roter Pfeil auf Play-Dialog-Button in der Toolbar.

## Panel-Aufbau

Das Preview-Panel ist in drei Bereiche geteilt:

```
┌─────────────────────────────────────────────────────┐
│  [Culture: de]          [Stop]  [Advance]            │  ← Steuerleiste
├─────────────────────────────────────────────────────┤
│                                                     │
│  [Portrait]  Wächter                                │  ← Dialog-Display
│              "Halt! Wer bist du?"                   │
│                                                     │
│  [ Ein Freund des Königs. ]                         │
│  [ Das geht dich nichts an. ]                       │
├─────────────────────────────────────────────────────┤
│  Tags:  [ ] Story.Found.Codex   [x] Player.Alert    │  ← Preview-State
│  Vars:  AngerLevel = 0  [Edit]                      │
│  Log:   Entry → SayLine_Guard_01                    │
└─────────────────────────────────────────────────────┘
```

## Steuerungs-Referenz

| Button | Wirkung |
| --- | --- |
| **Play** | Preview starten |
| **Stop** | Preview beenden, State zurücksetzen |
| **Advance** | Im „Warten auf Klick"-Modus: nächsten Node ansteuern |
| **Skip Typewriter** | Text sofort vollständig anzeigen |
| **Choice-Buttons** | Im PlayerChoice-Modus: Auswahl treffen |

## Was im Preview funktioniert

| Feature | Verhalten |
| --- | --- |
| SayLine | Text mit Typewriter-Effekt, Sprecher-Portrait sichtbar |
| Voice-Assets | Werden 2D abgespielt |
| Babel-Synthese | Wird ausgelöst (mit aktiver Culture) |
| PlayerChoice | Klickbare Buttons |
| Branch | Wertet Requirements gegen den Preview-State aus |
| RandomLine | Wählt zufällig |
| Wait | Wartet auf Advance-Klick oder Timer |
| Link / SubGraph | Funktioniert wie in Produktion |
| Set Variable | Schreibt in den Preview-State |
| Fire Event | Wird geloggt (kein echter Blueprint-Call) |
| Camera / Animation / Apply Effect | Geloggt, **nicht echt ausgeführt** (keine Welt vorhanden) |

## GAS-State simulieren

Der Preview-Runner scannt die im Asset verwendeten Tags, Attribute und Abilities und listet sie im Preview-State-Bereich auf. Du kannst sie **manuell an- und ausschalten**, bevor oder während der Dialog läuft:

- **Tags** – Checkbox-Liste. Aktivierte Tags simulieren das Vorhandensein am Spieler-ASC.
- **Attributes** – Zahlenfelder. Ändere den simulierten Wert (z.B. Health auf 10 setzen).
- **Abilities** – Checkbox-Liste. Simuliert, ob der Spieler eine Ability besitzt.

Sobald du einen Wert änderst, werden Requirement-Checks von Branch- und Choice-Sub-Nodes live neu ausgewertet.

> 📸 **Bild-Platzhalter:** `preview-runner-gas-state.png` — Preview-State-Bereich mit Tags-Liste und einem aktivierten Tag.
> *Setup:* Preview-State unten aufgeklappt. Tags-Sektion: `Story.Found.Codex` (Checkbox deaktiviert), `Player.Alert` (Checkbox aktiviert, blaues Häkchen). Darunter: Attributes mit leerem Health-Feld = 100. Roter Pfeil auf die Tag-Checkboxen.

## Variablen live setzen

Der Preview-State-Bereich zeigt alle Dialogue-Scope-Variablen mit ihrem aktuellen Wert. Klick auf den Wert öffnet ein Eingabefeld – ändere ihn und drücke Enter.

Nützlich um:
- Bestimmte Branches direkt anzusteuern (AngerLevel auf 3 setzen, ohne 3x zu provozieren)
- Zweige zu testen, die im normalen Durchlauf selten aufgerufen werden
- Default-Werte für den ersten Testdurchlauf zu überschreiben

> 📸 **Bild-Platzhalter:** `preview-runner-variable-edit.png` — Preview-State, Variable „AngerLevel" im Bearbeitungsmodus (Eingabefeld aktiv mit Wert „3").
> *Setup:* Preview läuft oder pausiert. Variable `AngerLevel` angeklickt, Eingabefeld offen mit Wert „3". Roter Pfeil auf das Eingabefeld. Im Dialog-Display oben sieht man den aktuellen SayLine-Text.

## Culture-Wechsel

Das Culture-Dropdown oben im Preview-Panel schaltet die aktive Sprache um:

1. SayLine-Voice-Assets werden aus dem `VoicePerCulture`-Slot der neuen Culture neu aufgelöst.
2. `FText`-Werte aktualisieren sich automatisch über UEs Lokalisierungssystem.
3. Babel-Synthese wird mit dem neuen Text neu angestoßen.

> 📸 **Bild-Platzhalter:** `preview-runner-culture-switch.png` — Culture-Dropdown offen mit „de", „en", „fr" als Optionen.
> *Setup:* Preview-Panel, Culture-Dropdown aufgeklappt. Drei Optionen sichtbar, „de" aktuell aktiv. Roter Pfeil auf das Dropdown. Im Dialog-Display darunter sieht man den deutschen Text der aktuellen SayLine.

## Event-Log

Am unteren Rand des Preview-State-Bereichs zeigt ein zeitgeordneter Log:

- Node-Übergänge: *„Entry → SayLine_Guard_01 → PlayerChoice_01"*
- Variable-Änderungen: *„AngerLevel: 0 → 1"*
- Fire-Event-Auslösungen: *„Event: Dialogue.Event.QuestUpdated"*
- Fehler: *„Requirement failed: CheckDialogueVariable(AngerLevel >= 3)"*

## Wann du stattdessen PIE nutzen solltest

Der Preview-Runner eignet sich nicht für alles:

| Szenario | Besser mit |
| --- | --- |
| Dialog braucht echte AI-Zustände | PIE + Debugger |
| Kamera-Schwenks und Animationen testen | PIE |
| GameplayEffects müssen sich echt auswirken | PIE |
| Multiplayer-Pfade testen | PIE |

Für alles andere – Texte, Choices, Variablen, Audio – ist der Preview-Runner schneller.

Weiter: [Debugger & Breakpoints →](debugger.md)
