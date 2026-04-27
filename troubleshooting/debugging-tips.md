---
description: Werkzeuge und Methoden für systematisches Debugging von MayDialogue-Problemen.
---

# Debug-Tipps

Wenn die Symptom-Liste in den häufigen Problemen nicht weiterhilft, gehe systematisch vor. Diese Seite zeigt dir die verfügbaren Werkzeuge und wie du sie sinnvoll einsetzt.

---

## Output-Log: Verbose-Logging aktivieren

Das schnellste Werkzeug bei Problemen ist der Output-Log mit erweiterter Ausgabe.

**Im laufenden Editor / PIE via Konsole:**

Drücke `~` (Tilde) um die UE-Konsole zu öffnen und gib ein:

```ini
log LogMayDialogue Verbose
```

Ab sofort gibt MayDialogue detaillierte Meldungen aus — Node-Übergänge, Participant-Auflösungen, Requirement-Ergebnisse.

**Dauerhaft per DefaultEngine.ini:**

```ini
[Core.Log]
LogMayDialogue=Verbose
```

**Output-Log öffnen:** `Window → Developer Tools → Output Log`. Filtere den Log auf `LogMayDialogue`, um nur Plugin-Meldungen zu sehen.

> 📸 **Bild-Platzhalter:** `debug-output-log-filter.png` — UE Output-Log mit aktivem Filter auf LogMayDialogue.
> *Setup:* PIE aktiv, Dialog ausgelöst. Output-Log-Fenster sichtbar. Im Suchfeld steht `LogMayDialogue`. Sichtbare Log-Zeilen zeigen den Dialog-Ablauf: `[MayDialogue] StartDialogue: Asset DA_GuardConversation — Instance created`, `[MayDialogue] Node executed: SayLine_0 (Speaker: Guard)`, `[MayDialogue] Presenting choices: 2 visible, 0 hidden`.

### Wichtige Log-Meldungen und ihre Bedeutung

| Meldung | Bedeutung |
| --- | --- |
| `StartDialogue: Asset X has no entry point` | Compile-Fehler oder Entry-Node fehlt |
| `Cannot resolve participant with tag X` | Speaker-Tag matcht keinen Actor |
| `Async node X timed out` | Wait-Node / PlayAnimation wartet zu lange |
| `Variable X not declared in asset` | SetVariable auf nicht-deklariertem Namen |
| `Choice index out of bounds` | Programmatischer `SelectChoice`-Aufruf mit ungültigem Index |
| `Instance cleanup: aborted` | Dialog wurde extern abgebrochen |

---

## Dialog-Debugger: Breakpoints und Step-Modus

Der Dialog-Debugger ist der mächtigste Weg, um Ablauf-Probleme zu isolieren. Er funktioniert während PIE.

**Breakpoint setzen:**

1. Dialog-Asset öffnen.
2. Rechtsklick auf den verdächtigen Node → **Toggle Breakpoint**.
3. PIE starten, Dialog auslösen.
4. Beim Treffer: der Node leuchtet auf, der Debugger-Tab wird aktiv.
5. **Step Over** — führt den nächsten Node aus.
6. **Step Into** — betritt Sub-Graphen oder verlinkte Assets (Cross-Asset, wenn verfügbar).
7. **Continue** — läuft weiter bis zum nächsten Breakpoint.

> 📸 **Bild-Platzhalter:** `debug-debugger-active.png` — Dialog-Debugger-Tab während eines aktiven Breakpoints.
> *Setup:* Dialog-Asset offen, PIE aktiv. Debugger-Tab sichtbar. Ein SayLine-Node leuchtet gelb (aktiver Zustand). Im Debugger-Tab rechts: Variable-Watch-Liste mit `bMetGuard = false`, `ChosenGreeting = ""`. Buttons Step Over, Step Into, Continue sind sichtbar und aktiv.

**Variable-Watch nutzen:**

Im Watch-Panel kannst du Variablen live sehen und manuell überschreiben. Nützlich, um zu prüfen:

- Schlägt ein Requirement fehl, weil der Bool-Wert `false` ist, obwohl er `true` sein sollte?
- Welcher Branch-Pfad wird genommen?

Klicke auf einen Variablen-Wert im Watch-Panel und gib einen neuen Wert ein — der nächste Node-Schritt verwendet ihn.

---

## Preview-Runner: Iteration ohne PIE

Der Preview-Runner spielt den Dialog direkt im Asset-Editor ab — kein PIE-Start, keine Participant-Komponenten nötig.

**Wann nützt er am meisten:**

- Schnelles Testen von Text, Reihenfolge und Branch-Logik.
- Simulieren von Tag-States ohne GAS-Setup.
- Überprüfen, ob ein Problem im Asset selbst liegt oder im Spiel-Setup.

**Diagnose-Regel:**

```
Läuft der Dialog im Preview-Runner korrekt?
  │
  ├─ Ja → Problem liegt im PIE-Setup (Participants, Tags, Widget, Input)
  └─ Nein → Problem liegt im Asset selbst (Nodes, Links, Requirements)
```

> 📸 **Bild-Platzhalter:** `debug-preview-runner.png` — Preview-Runner-Panel mit aktivem Dialog.
> *Setup:* Dialog-Asset offen. Preview-Runner-Tab aktiv. Sichtbar: SayLine-Text „Halt! Wer bist du?", Sprecher-Name „Wächter", zwei Choice-Buttons darunter. Links im Panel: simulierte Tag-Liste mit `Story.Found.Codex = true`. Der erste Choice-Button ist grün (Requirement erfüllt), der zweite grau (FailedButVisible).

**Simulated Tags im Preview-Runner setzen:**

Im Preview-Runner-Panel gibt es einen Tags-Bereich. Füge dort Gameplay-Tags hinzu, um Requirements zu simulieren, ohne in PIE gehen zu müssen. So kannst du alle Branches testen, ohne echte GAS-Setups zu brauchen.

---

## Outline-Panel: Fehler schnell lokalisieren

Wenn der Compiler einen Fehler meldet, aber du nicht weißt, wo im Graph er ist:

1. Öffne das **Outline-Panel** (Tab im Asset-Editor).
2. Filtere nach Node-Typ (z. B. nur SayLines anzeigen).
3. Klicke auf Einträge — der Graph springt zur Node-Position.

Mit **Ctrl+F** (Find-in-Dialogue) kannst du nach Text, Speaker-Tags oder Kommentaren suchen. Nützlich in großen Graphen mit vielen Nodes.

> 📸 **Bild-Platzhalter:** `debug-outline-panel.png` — Outline-Panel mit Filter und Liste aller SayLine-Nodes.
> *Setup:* Großes Dialog-Asset offen (20+ Nodes). Outline-Tab aktiv. Filter-Dropdown auf „SayLine" eingestellt. Liste zeigt alle SayLines mit Sprecher-Tag und Preview des ersten Satzes. Ein Eintrag ist rot markiert (Validator-Fehler: fehlendes Voice-Asset).

---

## Fallback auf Slate-Widget

Wenn du nicht weißt, ob das Problem im UMG-Widget oder tiefer liegt:

1. **Project Settings → MayDialogue → UI** → `bUseSlateDialogueWidget = true`, `DefaultDialogueWidgetClass = None`.
2. Editor neu starten, PIE starten, Dialog auslösen.

| Ergebnis | Bedeutung |
| --- | --- |
| Dialog im Slate-Widget sichtbar | Bug liegt im UMG-Widget-Setup |
| Dialog auch im Slate-Widget unsichtbar | Bug liegt tiefer (Runtime, Participant, Subsystem) |

---

## Isolations-Test: Halbieren

Wenn ein großes Asset kaputt ist und du nicht weißt, wo:

1. Erstelle eine **Kopie** des Assets (`Rechtsklick → Duplicate`).
2. Lösche die hintere Hälfte der Nodes in der Kopie.
3. Compile und teste.
4. Fehler weg? Das Problem lag in der gelöschten Hälfte.
5. Fehler noch da? Das Problem liegt in der verbliebenen Hälfte.
6. Wiederhole mit der verbleibenden Hälfte — **Binär-Suche auf Node-Ebene**.

---

## Crash-Analyse

Bei einem echten UE-Crash:

1. Crash-Log öffnen: `Saved/Logs/` im Projekt-Ordner, neueste `.log`-Datei.
2. Stack-Trace nach `UMayDialogueInstance`, `UMayDialogueNode_`, `UMayDialogueParticipant` durchsuchen.
3. Null-Dereference? → Prüfe Instance-Lebensdauer (zu früh zerstört?).
4. Array-Out-of-Bounds? → Choice-Index-Validierung prüfen (programmatischer `SelectChoice`-Aufruf).

Beim Melden eines Bugs: Repro-Steps + Log-Auszug (nur relevante Zeilen) + Plugin-Version angeben.

---

## Performance-Probleme

Wenn der Dialog „ruckelt" oder spürbar langsam ist:

| Symptom | Was prüfen |
| --- | --- |
| Typewriter zu schnell oder zu langsam | `TypewriterCharsPerSecond` in den Speaker-Settings |
| Portrait-Textur kostet viel | Texture-Size im Asset-Detail prüfen (max. 512×512 empfohlen) |
| Babel erzeugt viele Sounds auf einmal | Anzahl BlipSounds-Samples reduzieren |
| Kamera-Blend ruckelt | `BlendDuration` erhöhen, parallele Blends reduzieren |
