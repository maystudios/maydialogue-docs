# Roadmap

Offene Arbeits-Posten und Vision für zukünftige Versionen.

## Nahfristig (nächste Releases)

### Bug-Fixes (aus dem Backlog)

* **Widget-Lifetime**: Viewport-Widget sauber aus- und wiederbinden nach Level-Wechsel.
* **Wait-Node Abort-Cleanup**: Timer sauber abbrechen.
* **PlayAnimation Abort-Cleanup**: Montage-Delegates unbinden.
* **Input-Mode-Restore**: Vorherigen Modus cachen statt hart-auf-GameOnly.
* **FailBehavior in ExecuteNode**: Requirement-Ergebnis + FailBehavior konsequent auswerten.

### Feature-Lücken

* **`bOverride2D` auf SayLine / PlaySound**: Node-Level 2D-Override freischalten.
* **SayLine `VolumeMultiplier` / `PitchMultiplier`**: Analog PlaySound.
* **SetVariable Tag-Typ**: UI-Pfad fertigstellen.
* **SetVariable Participant-Scope**: Schreib-Pfad auf PersistentMemory.
* **Wait-Node Condition-Modus**: Polling-Requirement-Check.
* **Live-Requirement-Pills im Preview**: Choice/Branch-Pills in Echtzeit einfärben.
* **Cross-Asset-Step-Into im Debugger**.
* **QuickSave-Helper**: Finalisierung der API.

## Mittelfristig

### UMG-Themes

Drei Starter-Themes ausliefern:

* **Horror**: dunkel, blutrot, pixelig.
* **Visual Novel**: großer Portrait-Bereich, weiche Animation.
* **RPG**: klassische Dialog-Box mit Name-Plate.

### Bridge-Erweiterung

* `OnNodeReached`-Delegate auf der Bridge (nicht nur Instance).
* Read-API für Scope-Stack-Info.
* Write-API für Save-/Restore-Snapshots der Instance-Variables.

### Babel-Polish

* Sample-Caching für Procedural-Blips.
* Parametrisierbare Phoneme-Prosodie-Kurven.
* Pro-Speaker-Profile als DataTable für schnelle Varianz.

### Editor-Features

* **Live-Validation** als optionaler Opt-in (asynchroner Debounce-Check).
* **Minimap** als eigener Tab (derzeit Outline bevorzugt, aber Minimap wäre nice-to-have).
* **Cross-Asset-Navigation bei Link-Nodes**: Editor öffnet Ziel-Asset automatisch bei Step-Into.

### Replikation

* `ClientUpdateConversation`-RPC mit net-serialisierbarer Message.
* Net-safe Struct-Variants für `FMayDialogueMessage` und `FMayDialogueChoiceEntry`.
* Verify Multiplayer-Pfade (Host + Client) über Integration-Tests.

## Langfristig

### DataTable-Integration

Speaker-Templates als DataTable-Rows:

```
Speaker_Guard
├── DisplayName: "Wächter"
├── Portrait: P_Guard
├── NodeColor: #8B0000
├── SoundClass: SC_VoiceGuard
├── Attenuation: Att_NPC
└── BabelProfile: BP_Babel_Guard
```

Assets referenzieren die Row statt eigenen Speaker-Struct zu halten. Änderungen an der Row propagieren projektweit.

### Visual-Preview-Widget

Im Asset-Editor ein WYSIWYG-Widget, das SayLines mit Portrait, Emotion-Tag-Visualisierung und Typewriter rendert — ohne Preview-Runner starten zu müssen.

### Import / Export

* **Import aus Articy, Yarn-Spinner, Twine**: Dialog-Scripts nach MayDialogue portieren.
* **Export als JSON**: für externe Script-Analyse und Übersetzungs-Pipelines.

### Node-Marketplace

Community-contributed Nodes (eigene Requirement-/SideEffect-/Action-Klassen) als Share-Paket.

## Nicht-Ziele

Was wir **bewusst nicht** tun werden (siehe auch [Non-Goals im README](../../README.md#ist-maydialogue-das-richtige-für-dich)):

* Kein Quest-System.
* Kein Cutscene-Sequencer.
* Kein Voice-Recording-Tool.
* Keine Multiplayer-UX (Voting, geteilte Sessions).
* Kein Live-Collaboration-Editor.

Das Plugin bleibt **fokussiert**. Features, die den Dialog-Scope sprengen, gehören in separate Werkzeuge.

## Community-Input

Wenn du Features auf dieser Roadmap priorisiert haben willst: Issue im Tracker öffnen mit Begründung (welche konkrete Projekt-Notwendigkeit?). Die Roadmap ist nicht in Stein gemeißelt.
