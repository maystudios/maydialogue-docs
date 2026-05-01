---
description: Aktuelle Einschränkungen und empfohlene Vorgehensweisen.
---

# Bekannte Issues

Stand: Plugin-Version **1.0.0**. Diese Liste wird mit jeder Release aktualisiert.

{% hint style="info" %}
Wenn du auf ein Problem stößt, das hier nicht aufgeführt ist, prüfe zuerst die [Debug-Tipps](debugging-tips.md). Für neue Meldungen: Repro-Steps, Log-Auszug und Plugin-Version angeben.
{% endhint %}

---

## Offene Issues

### Widget & UI

| Problem | Empfohlener Workaround |
| --- | --- |
| UMG Starter-Themes (Horror, VN, RPG) noch nicht mitgeliefert. | Eigenes Theme aufsetzen (siehe [UI-Architektur](../ui/umg-architecture.md)). |

### Audio

| Problem | Empfohlener Workaround |
| --- | --- |
| Babel-Synthesizer: Audio-Qualität und Konfigurierbarkeit (Per-Speaker-Depth, Biquad-Filter) in Überarbeitung. | Eigene `BlipSounds` zuweisen statt den prozeduralen Modus zu nutzen. |
| Babel Global Off: Flag `bEnableBabelVoice` vorhanden; vollständige Stummschaltung ohne Voice-Asset in Kombination mit Typewriter noch nicht abschließend verifiziert. | Eigenes Projekt testen und ggf. `BlipSounds`-Array leer lassen. |

### Kamera

| Problem | Empfohlener Workaround |
| --- | --- |
| CameraFocus FOV-Restore läuft beim Instance-Cleanup global; das Original-FOV wird nicht pro Node gecacht. | Pro Dialog nur ein einzelnes FOV-Override verwenden. |

### Editor

| Problem | Empfohlener Workaround |
| --- | --- |
| Zyklus-Prüfung: Der Compiler erkennt Zyklen zuverlässig, das Schema lehnt sie beim Verbinden jedoch noch nicht in Echtzeit ab. | Beim Verbinden von Knot-Chains manuell auf Zyklen achten; der Compiler gibt beim Speichern einen Fehler aus. |
| Live-Validation ist standardmäßig deaktiviert (Re-Entry-Problem). | Compile-Button nach jeder Änderung drücken. |
| Minimap-Widget existiert im Code, ist aber keinem eigenen Editor-Tab zugeordnet. | Outline-Panel als Alternative nutzen. |

---

## Erledigte Issues

Diese Punkte galten früher als offen und sind in v1.0 behoben:

- Widget überlebt Level-Teardown nicht mehr (Subsystem-Deinitialize räumt Slate- und UMG-Widget korrekt auf).
- Wait-Timer bei Dialog-Abort: AsyncState_Wait bereinigt alle Timer in `Cleanup()`.
- PlayAnimation-Montage-End-Delegate bleibt nach Abort gebunden: `AsyncState_PlayAnimation::Cleanup` ersetzt den Delegate.
- `EMayDialogueNodeFailBehavior` wird in `ExecuteNode` konsequent ausgewertet.
- Wait-Node Condition-Modus (Polling) implementiert: `WaitCondition` + `ConditionCheckInterval` in `MayDialogueNode_Wait.h`.
- SetVariable schreibt jetzt Dialogue- und Participant-Scope über die `Scope`-Property.
- SetVariable unterstützt den Tag-Typ (`TagValue`-Branch) vollständig.
- Node-Level 2D-Override: `NodeAudioMode` (tri-state Enum) ersetzt das alte `bOverride2D`-Flag auf SayLine und PlaySound.
- `VolumeMultiplier` / `PitchMultiplier` auf SayLine verfügbar.
- Choice-Tag-Binding (ChoiceTags) auf dem Choice-Sub-Node implementiert; `OnChoiceMade`-Delegate übergibt ChoiceTags.
- QuickSave-Helper (`MayDialogueSaveHelper`) vollständig implementiert.
- Input-Mode-Restore: Beide UI-Pfade (Slate + UMG) erkennen den vorherigen Input-Mode zuverlässig (`UIOnly` via `GameViewportClient::IgnoreInput()`, `GameAndUI` vs `GameOnly` via Cursor-State) und stellen ihn nach Dialog-Ende exakt wieder her.
- Rich-Text-Decorators `<color>` und `<b>` funktional.
- Komponenten-basierte UMG-Widget-Variante (DialogFrame, Speaker, Text, ChoiceButton, ChoiceList, SkipButton) mitgeliefert.
- ApplyEffect als Action-Node implementiert.
- Debugger Step-Into / Step-Out implementiert.
- Participant `DefaultDialogue`, `SetActiveDialogue()`, `GetActiveDialogue()`, `StartDefaultDialogue()` verfügbar.
- Participant `AttenuationOverride` (AdvancedDisplay) verfügbar.
- 3-Level Audio-Settings (Plugin → Speaker → Node).
- Audio-Playback im Editor-Preview mit Culture-Auswahl.
- Preview-Runner und FindResults als eigene Tabs.
- Auto-Layout-Button (Sugiyama-Algorithmus).
- Knot-Zyklus-Detection im Slate-Widget.
- Compiler-ResolveKnotChain VisitedSet.
- Compiler Choice-Target GUID-Invalidierung.
- Unbegrenzte Immediate-Transitions gefixt (MaxHops + VisitedNodes).
- Random-Line State per-Instance (nicht mehr auf Asset-Node).
- Choice-Timeout-State bei neuem `PresentChoices` gecleart.
- Subsystem-Deinitialize räumt abgeschlossene Dialoge auf.
- Validator prüft async-Nodes ohne Continuation.
- Failed Dialogue-Start: Entry-Point-Check vor Instance-Erzeugung.

---

## Neues Issue melden

Nutze den GitHub-Issues-Tracker oder melde dich im Support-Kanal. Bitte immer mitliefern:

1. **Repro-Steps** — Was klicken, was erwarten, was passiert.
2. **Log-Auszug** — Nur die relevanten Zeilen aus `Saved/Logs/`.
3. **Plugin-Version** — Sichtbar in `MayDialogue.uplugin` unter `VersionName`.
