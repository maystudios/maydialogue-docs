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

### In v1.0 behoben

Folgende Punkte aus der [Known-Issues-Liste](../troubleshooting/known-issues.md) wurden in v1.0 geschlossen:

- ~~**Widget-Lifetime**~~: ✅ Behoben — Subsystem räumt Slate- und UMG-Auto-Widgets beim Deinitialize auf.
- ~~**Wait-Node Abort-Cleanup**~~: ✅ Behoben — `AsyncState_Wait::Cleanup` stoppt alle Timer; Abort ruft `CleanupPendingAsyncNodes` auf.
- ~~**PlayAnimation Abort-Cleanup**~~: ✅ Behoben — `AsyncState_PlayAnimation::Cleanup` überschreibt den Montage-End-Delegate sauber.
- ~~**Input-Mode-Restore (Slate)**~~: ✅ Behoben für den Slate-Pfad — `CachedPreviousInputMode` im Widget gecacht.
- ~~**FailBehavior in ExecuteNode**~~: ✅ Behoben — Node-Base und `Instance::ContinueToNode` werten `FailBehavior` konsequent aus.
- ~~**`bOverride2D` auf SayLine / PlaySound**~~: ✅ Ersetzt durch den Tri-State `NodeAudioMode` auf beiden Node-Typen.
- ~~**SayLine `VolumeMultiplier` / `PitchMultiplier`**~~: ✅ Beide Felder sind jetzt auf SayLine verfügbar.
- ~~**SetVariable Tag-Typ**~~: ✅ Tag-Typ-Variablen werden in SetVariable unterstützt.
- ~~**SetVariable Participant-Scope**~~: ✅ `Scope = Participant` + `TargetParticipantTag` implementiert.
- ~~**Wait-Node Condition-Modus**~~: ✅ `WaitCondition` + `ConditionCheckInterval` implementiert und im AsyncState gepollt.
- ~~**QuickSave-Helper**~~: ✅ `MayDialogueSaveHelper` bietet QuickSaveToSlot / QuickLoadFromSlot / DeleteSlot / DoesSlotExist.
- ~~**Dialog-Events → GameplayCues**~~: ✅ `LifecycleCueBindings` in den Settings, Subsystem leitet weiter.
- ~~**Choice-Tag-Binding extern**~~: ✅ `OnChoiceMade` enthält `ChoiceIndex` und `ChoiceTags`.
- ~~**Bridge Read/Write API**~~: ✅ Vollständige API auf `IMayDialogueBridge` und Blueprint-Wrapper am Subsystem.

### Noch offen

- **Input-Mode-Restore (UMG-Pfad)**: `UMayDialogueWidget` besitzt noch kein `ApplyDialogueInputMode` / `RestoreGameInputMode`.
- **Live-Requirement-Pills im Preview**: Choice- und Branch-Pills im Preview-Runner sollen Requirement-Status in Echtzeit einfärben.
- **Cross-Asset Step-Into im Debugger**: Debugger folgt dem Flow in verlinkte Assets.
- **Subsystem-level `OnAnyVariableChanged`**: Per-Instance-Binding funktioniert; globale Weiterleitung fehlt noch.

---

## Mittelfristig: Qualität und Komfort

### UMG Starter-Themes (v1.1)

Drei fertige Widget-Template-Sets — nicht in v1.0 enthalten, geplant für v1.1 als separates Content-Add-on:

- **Horror**: Dunkles Layout, rote Akzente, pixeliger Font-Stil.
- **Visual Novel**: Großer Portrait-Bereich, weiche Ein-/Ausblend-Animation.
- **RPG**: Klassische Dialogbox mit Name-Plate und Icon-Slot.

### Bridge-Erweiterung

- `OnAnyVariableChanged` global forward on Subsystem (per-instance binding already works).
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
