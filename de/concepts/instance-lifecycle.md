---
description: Was passiert von Start bis Exit — und wie der Cleanup funktioniert.
---

# Instance & Lifecycle

Das Dialog-Asset ist die Blaupause. Das **laufende Gespräch** ist eine Instanz — ein kurzlebiges Objekt, das für genau einen Dialog existiert. Dieses Kapitel zeigt, welche Phasen eine Instanz durchläuft und was du an welchem Punkt tun kannst.

## Der Lebenszyklus auf einen Blick

```text
[Kein Dialog]
      │
      │  StartDialogue()
      ▼
   [Aktiv]  ──── SayLine bereit ──→  [Wartet auf Advance]
      │                                      │
      │  PlayerChoice bereit                 │  AdvanceDialogue()
      ▼                                      │
[Wartet auf Choice] ◄──────────────────────┘
      │
      │  SelectChoice(Index)
      ▼
   [Aktiv]  ──── Exit-Node erreicht ──→  [Completed / Failed]
      │
      │  AbortDialogue() von außen
      ▼
   [Aborted]
```

Das Plugin verwendet ein robustes Lifecycle-System, das sicherstellt, dass eine Instanz immer sauber aufräumt — unabhängig davon, wie sie endet.

> 📸 **Bild-Platzhalter:** `lifecycle-state-diagram.png` — Zustandsdiagramm als Grafik (kein Mermaid-Code).
> *Setup:* Handgezeichneter oder sauber gestalteter Flussplan mit sechs Zuständen als Kästen: Inaktiv, Aktiv, WartendAufAdvance, WartendAufChoice, Completed, Aborted. Pfeile beschriftet mit den auslösenden Aktionen (StartDialogue, ReceiveMessage, AdvanceDialogue, SelectChoice, Exit-Node, AbortDialogue). Completed und Aborted haben einen dicken Rahmen (Endzustände).

## Startphase: Was beim StartDialogue passiert

Wenn du `StartDialogue(Asset, Instigator, Target)` aufrufst, laufen fünf Schritte ab:

1. **Pre-Flight-Check** — Hat das Asset einen Entry-Node? Ist bereits ein anderer Dialog aktiv?
2. **Instanz erzeugen** — Eine neue Instanz wird erstellt und an die Welt gebunden.
3. **Participants auflösen** — Die Sprecher-Tags im Asset werden mit den Participant-Komponenten im Level abgeglichen.
4. **OnDialogueStarted feuern** — Dein Code kann jetzt reagieren.
5. **Entry-Node ansteuern** — Der erste Node wird ausgeführt.

> 📸 **Bild-Platzhalter:** `blueprint-on-dialogue-started.png` — Blueprint-Graph, der auf OnDialogueStarted reagiert.
> *Setup:* BP-Graph eines GameMode- oder HUD-Actors. Event `OnDialogueStarted` (Delegate-Bind) → `Set Input Mode UI Only` → `Show Mouse Cursor = true`. Alle Pins beschriftet. Deutlich: kein manuelles Widget-Erstellen nötig — das Plugin macht das selbst.

## Node-Ausführung: Was jeder Node zurückgibt

Jeder Node gibt nach seiner Ausführung einen Anweisung zurück, wie der Dialog weiterlaufen soll:

| Rückgabe | Was passiert |
| --- | --- |
| Advance (Next-Node) | Direkt zum angegebenen nächsten Node springen. |
| Pause + Choices präsentieren | Instanz wartet. Widget zeigt Choice-Buttons. |
| Zurück zum letzten Choice | Springt zum zuletzt präsentierten PlayerChoice zurück. |
| Zurück zum Dialog-Start | Springt an den Entry zurück. |
| Abort | Dialog endet als Aborted. |

Als Nutzer musst du diese Mechanik nicht direkt ansteuern — die Nodes des Plugins verwenden sie intern korrekt.

## Requirements: Wann wird ein Node ausgeführt?

Bevor ein Node ausgeführt wird, prüft die Instanz seine Requirements (falls vorhanden).

| Ergebnis | Was passiert |
| --- | --- |
| Passed | Node wird normal ausgeführt. |
| FailedButVisible | Node-Inhalt wird dem Spieler gezeigt, ist aber nicht wählbar. |
| FailedAndHidden | Node erscheint gar nicht. |

Wenn ein Requirement fehlschlägt, entscheidet das `FailBehavior`-Feld des Nodes, ob der Dialog überspringt oder abbricht.

## Choices: Der interaktive Schritt

Ein `PlayerChoice`-Node baut seine Liste in drei Schritten:

1. **Bauen** — Für jede Choice werden Requirements geprüft und ein Eintrag mit Availability, Text und Tags erstellt.
2. **Filtern** — `FailedAndHidden`-Choices werden entfernt.
3. **Präsentieren** — Die Instanz wartet. Das Widget zeigt die verbleibenden Choices.

Wenn der Spieler klickt → `SelectChoice(Index)`:

1. Die Availability wird erneut geprüft (Variablen könnten sich geändert haben).
2. SideEffects der Choice werden ausgeführt.
3. Der Dialog springt zum Ziel-Node der Choice.

> 📸 **Bild-Platzhalter:** `choices-ingame-widget.png` — In-Game-Widget mit aktiven Choice-Buttons.
> *Setup:* PIE-Modus, Dialog aktiv. Widget zeigt unten drei Buttons nebeneinander. Button 1: „Ein Freund des Königs." (weiß, klickbar). Button 2: „Das geht dich nichts an." (ausgegraut, Tooltip: „Nicht verfügbar"). Button 3 ist nicht sichtbar (FailedAndHidden gefiltert). Hintergrund: das Level mit dem Wächter-NPC.

## Async-Nodes: Wenn ein Node wartet

Manche Nodes pausieren den Dialog, bis ein externes Event eintrifft:

- **Wait** — wartet auf einen Timer oder einen Event-Tag.
- **PlayAnimation** — wartet optional bis eine Montage endet.
- **CameraFocus** — wartet auf das Ende der Blend-Zeit.

Der Dialog bleibt in diesem Zustand aktiv, aber keine neuen Nodes werden ausgeführt, bis das Event feuert.

{% hint style="warning" %}
Wenn ein async Node stecken bleibt (Event-Tag feuert nie, Montage endet nie), bleibt die Instanz hängen. Der Validator im Editor warnt, wenn ein async Node keine klare Fortsetzung hat.
{% endhint %}

## Links & Sub-Dialoge

Ein `Link`-Node oder `SubGraph`-Node verzweigt in einen anderen Dialog oder einen eingebetteten Unter-Graph. Das Plugin merkt sich dabei den Rücksprung-Punkt.

Wenn der verlinkte Dialog endet:
- Gibt es einen Rücksprung-Punkt → der Dialog setzt dort fort.
- Gibt es keinen → der Dialog endet normal als Completed.

So können Dialog-Fragmente beliebig tief verschachtelt werden, ohne dass du den Rücksprung selbst verwalten musst.

## Endphase: Drei Wege zum Ende

### Completed

Der Dialog erreicht einen `Exit`-Node mit Status `Completed`:

1. Alle wartenden Timer und Event-Listener werden getrennt.
2. Laufende Voice-Wiedergabe wird gestoppt.
3. Kamera wird auf den Ursprung zurückgeblendet (wenn ein CameraFocus aktiv war).
4. `OnDialogueEnded` feuert mit Status `Completed`.
5. Das Subsystem entfernt die Instanz am Frame-Ende.

### Failed

Wie Completed, aber der Exit-Node hat Status `Failed`. Nützlich um im aufrufenden Code zwischen „abgeschlossen" und „fehlgeschlagen" zu unterscheiden.

### Aborted

Ausgelöst von außen: neuer Dialog startet, Level-Wechsel, oder dein Code ruft `AbortDialogue()` auf. Gleicher Cleanup wie bei Completed, aber Status = `Aborted`.

> 📸 **Bild-Platzhalter:** `exit-node-details.png` — Details-Panel eines Exit-Nodes.
> *Setup:* Exit-Node im Graph ausgewählt. Details-Panel rechts zeigt: `ExitStatus = Completed` (Dropdown). Kein weiterer Property-Eintrag. Der Node selbst ist als rote Kapsel im Graph sichtbar.

## Delegate-Hooks: Wo du dich einklinken kannst

| Delegate | Wann | Typischer Einsatz |
| --- | --- | --- |
| `OnDialogueStarted` | Beim Start | Input-Modus umschalten, Kamera vorbereiten |
| `OnMessageReceived` | Pro SayLine | Audio starten, Untertitel setzen, Animation triggern |
| `OnChoicesPresented` | Wenn Choices bereitstehen | Buttons rendern (falls eigenes Widget) |
| `OnChoiceMade` | Nach Auswahl | Analytics, Achievements |
| `OnVariableChanged` | Pro Variable-Mutation | Quest-System benachrichtigen |
| `OnDialogueEnded` | Beim Ende | Input-Modus zurücksetzen, Quest-Check auslösen |

Alle Delegates sind **Multicast** — mehrere Systeme können gleichzeitig lauschen.

> 📸 **Bild-Platzhalter:** `blueprint-on-dialogue-ended.png` — Blueprint-Graph für OnDialogueEnded.
> *Setup:* BP-Graph eines PlayerControllers. `Bind Event to OnDialogueEnded` → Event feuert, Parameter `ExitStatus` geprüft mit `Switch on EMayDialogueExitStatus` → Zweig Completed führt zu `Quest Check Complete`, Zweig Aborted führt zu `Restore Player Input`. Alle Pins sauber beschriftet.

## Zusammenfassung

- Eine Instanz entsteht beim Start und wird am Ende automatisch aufgeräumt — kein manuelles Verwalten nötig.
- Zustände: Aktiv, WartendAufAdvance, WartendAufChoice, Completed, Aborted, Failed.
- Async-Nodes pausieren den Dialog; der Cleanup trennt alle Listener sauber.
- `Link`/`SubGraph` macht verschachtelte Dialoge ohne manuellen Rücksprung-Code.
- Die Delegate-Hooks sind der Weg, wie externe Systeme (Quest, Audio, Analytics) reagieren.

Weiter: [Participants & Sprecher](participants-speakers.md).
