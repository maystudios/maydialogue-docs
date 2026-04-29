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

### Bug-Fixes (v1.0 wave, verified 2026-04-29)

The following issues from the [Known-Issues list](../troubleshooting/known-issues.md) have been resolved:

- ~~**Widget-Lifetime**~~: ✅ Fixed — Subsystem Deinitialize tears down Slate and UMG auto-widgets.
- ~~**Wait-Node Abort-Cleanup**~~: ✅ Fixed — `AsyncState_Wait::Cleanup` stops all timers; `AbortDialogue` calls `CleanupPendingAsyncNodes`.
- ~~**PlayAnimation Abort-Cleanup**~~: ✅ Fixed — `AsyncState_PlayAnimation::Cleanup` replaces Montage end-delegate with empty lambda.
- ~~**Input-Mode-Restore (Slate)**~~: ✅ Fixed for Slate path — `CachedPreviousInputMode` switch in `SMayDialogueWidget`. **UMG path still open.**
- ~~**FailBehavior in ExecuteNode**~~: ✅ Fixed — `Node_Base.cpp` and `Instance::ContinueToNode` both honor FailBehavior.

**Still open:**
- **Input-Mode-Restore (UMG path)**: `UMayDialogueWidget` has no `ApplyDialogueInputMode` / `RestoreGameInputMode` yet.

### Feature-Lücken (v1.0 wave)

The following feature gaps have been resolved:

- ~~**`bOverride2D` on SayLine / PlaySound**~~: ✅ Superseded by tri-state `NodeAudioMode` on both nodes.
- ~~**SayLine `VolumeMultiplier` / `PitchMultiplier`**~~: ✅ `SayLine.h` exposes both.
- ~~**SetVariable Tag-Typ**~~: ✅ `SetVariable.h::TagValue` implemented.
- ~~**SetVariable Participant-Scope**~~: ✅ `Scope = Dialogue|Participant` + `TargetParticipantTag` implemented.
- ~~**Wait-Node Condition-Modus**~~: ✅ `WaitCondition` + `ConditionCheckInterval` implemented and polled in AsyncState.
- ~~**QuickSave-Helper**~~: ✅ `MayDialogueSaveHelper.h` exposes QuickSaveToSlot / QuickLoadFromSlot / DeleteSlot / DoesSlotExist.
- ~~**Dialogue-Events → GameplayCues**~~: ✅ `LifecycleCueBindings` in Settings + Subsystem bridging.
- ~~**Choice-Tag-Binding external**~~: ✅ `OnChoiceMade` is 2-param (`ChoiceIndex`, `ChoiceTags`).
- ~~**Bridge Read/Write API**~~: ✅ Full API on `IMayDialogueBridge` and K2_* Blueprint mirrors on Subsystem.

**Still open:**
- **Live-Requirement-Pills im Preview**: Choice- und Branch-Pills im Preview-Runner in Echtzeit einfärben.
- **Cross-Asset Step-Into im Debugger**: Debugger folgt dem Flow in verlinkte Assets.
- **Subsystem-level `OnAnyVariableChanged`**: Per-instance binding works; global forward still missing.

---

## Mittelfristig: Qualität und Komfort

### UMG Starter-Themes (v1.1)

Three ready-to-use widget template sets — **not shipping in v1.0**, planned for v1.1 as a separate content add-on:

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
