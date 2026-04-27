---
description: Geplante Verbesserungen und langfristige Richtung des Plugins.
---

# Roadmap

Diese Seite zeigt, woran gearbeitet wird und wohin sich das Plugin entwickelt. Alle Punkte sind **geplant**, keine Termine sind festgelegt. Prioritäten können sich verschieben.

{% hint style="info" %}
Wenn du einen Punkt priorisiert haben möchtest: Lege ein Issue im Tracker deines Teams an und begründe den konkreten Projekt-Bedarf. Die Roadmap ist kein Versprechen — sie zeigt die aktuelle Bewertung.
{% endhint %}

---

## Kurzfristig: Offene Lücken schließen

### Bug-Fixes

Die folgenden Issues aus der [Known-Issues-Liste](../troubleshooting/known-issues.md) haben hohe Priorität:

- **Widget-Lifetime**: Viewport-Widget sauber aus- und wiederbinden nach Level-Wechsel.
- **Wait-Node Abort-Cleanup**: Timer sauber abbrechen, wenn ein Dialog abgebrochen wird.
- **PlayAnimation Abort-Cleanup**: Montage-Delegates korrekt aufräumen.
- **Input-Mode-Restore**: Vorherigen Input-Modus cachen statt hart auf `GameOnly` zu setzen.
- **FailBehavior in ExecuteNode**: Requirement-Ergebnis und FailBehavior konsequent auswerten.

### Feature-Lücken

- **`bOverride2D` auf SayLine / PlaySound**: Node-Level 2D-Override freischalten.
- **SayLine `VolumeMultiplier` / `PitchMultiplier`**: Analog zu PlaySound.
- **SetVariable Tag-Typ**: UI-Pfad für Tag-Variablen fertigstellen.
- **SetVariable Participant-Scope**: Direkter Schreibpfad auf persistenten Speicher.
- **Wait-Node Condition-Modus**: Polling-basierter Requirement-Check als dritter Wait-Modus.
- **Live-Requirement-Pills im Preview**: Choice- und Branch-Pills im Preview-Runner in Echtzeit einfärben.
- **Cross-Asset Step-Into im Debugger**: Debugger folgt dem Flow in verlinkte Assets.
- **QuickSave-Helper**: API-Implementierung finalisieren.

---

## Mittelfristig: Qualität und Komfort

### UMG Starter-Themes

Drei fertige Widget-Vorlagen zum sofortigen Einsatz:

- **Horror**: Dunkles Layout, rote Akzente, pixeliger Font-Stil.
- **Visual Novel**: Großer Portrait-Bereich, weiche Ein-/Ausblend-Animation.
- **RPG**: Klassische Dialogbox mit Name-Plate und Icon-Slot.

### Bridge-Erweiterung

- `OnNodeReached`-Delegate auf der Bridge (nicht nur auf der Instance).
- Read-API für aktuelle Scope-Infos.
- Write-API für Save-/Restore-Snapshots von Instance-Variablen — nützlich für Checkpoint-Systeme.

### Babel-Polish

- Sample-Caching für prozedurale Blips (Performance-Verbesserung).
- Parametrisierbare Phonem-Prosodie-Kurven für natürlichere Stimmprofile.
- Per-Speaker-Profile als DataTable für schnelle projektweite Varianz.

### Editor-Features

- **Live-Validation**: Optionaler asynchroner Debounce-Check nach Änderungen.
- **Minimap-Tab**: Eigener Tab neben dem Outline-Panel.
- **Cross-Asset-Navigation**: Editor öffnet Ziel-Asset automatisch, wenn man auf einen Link-Node klickt.

### Replikation

- `ClientUpdateConversation`-RPC mit net-serialisierbarer Message.
- Net-sichere Struct-Varianten für Dialog-Messages und Choice-Entries.
- Multiplayer-Pfade (Host + Client) über Integrations-Tests verifizieren.

> 📸 **Bild-Platzhalter:** `roadmap-theme-preview.png` — Drei Widget-Themes nebeneinander in PIE.
> *Setup:* Drei PIE-Fenster geöffnet, je eines mit Horror-, VN- und RPG-Theme. Dieselbe SayLine „Was willst du hier?" wird in allen drei Themes angezeigt. Sichtbar sind die Unterschiede in Hintergrundfarbe, Font, Portrait-Position und Button-Stil.

---

## Langfristig: Erweiterter Funktionsumfang

### DataTable-Integration für Speaker

Speaker-Definitionen als DataTable-Rows, die projektweite Änderungen an einem einzigen Ort ermöglichen:

```text
Speaker_Guard
├── DisplayName: "Wächter"
├── Portrait: P_Guard
├── NodeColor: #8B0000
├── SoundClass: SC_VoiceGuard
├── Attenuation: Att_NPC
└── BabelProfile: BP_Babel_Guard
```

Assets referenzieren die Row statt einen eigenen Speaker-Struct zu halten. Änderungen am Speaker propagieren automatisch in alle Assets.

### Visual-Preview-Widget

Ein WYSIWYG-Vorschau-Panel direkt im Asset-Editor: SayLines werden mit Portrait, Emotion-Tag-Visualisierung und Typewriter gerendert — ohne den Preview-Runner starten zu müssen.

### Import / Export

- **Import aus Articy, Yarn-Spinner, Twine**: Bestehende Dialog-Skripte nach MayDialogue portieren.
- **Export als JSON**: Für externe Script-Analyse, Übersetzungs-Pipelines und Qualitätssicherung.

### Node-Erweiterungen durch die Community

Ein strukturiertes Format für Community-contributed Nodes — eigene Requirement-, SideEffect- und Action-Klassen als verteilbares Share-Paket.

---

## Nicht-Ziele

Das Plugin bleibt auf seinen Kernbereich fokussiert. Folgende Themen sind bewusst ausgeschlossen:

- Kein Quest-System (separate Verantwortlichkeit).
- Kein Cutscene-Sequencer.
- Kein Voice-Recording- oder Voice-Casting-Tool.
- Keine Multiplayer-UX (geteilte Sessions, Voting).
- Kein Live-Collaboration-Editor.

Features, die den Dialog-Scope sprengen, gehören in separate Werkzeuge — damit das Plugin schlank und wartbar bleibt.
