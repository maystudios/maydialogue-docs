---
description: Offene Einschränkungen der aktuellen Beta-Version und empfohlene Workarounds.
---

# Bekannte Issues

Stand: Plugin-Version **0.1.0 (Beta)**. Diese Liste wird mit jeder Release aktualisiert.

{% hint style="info" %}
Wenn du auf ein Problem stößt, das hier nicht aufgeführt ist, prüfe zuerst die [Debug-Tipps](debugging-tips.md). Für neue Meldungen: Repro-Steps, Log-Auszug und Plugin-Version angeben.
{% endhint %}

---

## Offene Issues

### Widget & UI

| Problem | Empfohlener Workaround |
| --- | --- |
| Static-Widget überlebt Level-Teardown; bindet sich nach Level-Wechsel nicht neu. | `Subsystem → StopAllDialogues()` beim Level-Wechsel aufrufen, oder Widget manuell aus dem Viewport entfernen. |
| UMG Starter-Themes (Horror, VN, RPG) noch nicht mitgeliefert. | Eigenes Theme aufsetzen (siehe [UI → Themes](../ui/themes.md)). |

### Nodes & Runtime

| Problem | Empfohlener Workaround |
| --- | --- |
| Wait-Timer überlebt Dialog-Abort unter bestimmten Cleanup-Pfaden. | Wait-Dauer kurz halten; oder Timeout-Event feuern, um frühzeitig aufzuräumen. |
| PlayAnimation-Montage-End-Delegates bleiben bei Abort gebunden. | `bWaitForMontageEnd = false` setzen in risikoreichen Pfaden, oder Animation-End manuell verdrahten. |
| `EMayDialogueNodeFailBehavior` ist deklariert, wird aber in `ExecuteNode` noch nicht konsequent ausgewertet. | Requirements redundant auf Outputs legen, wenn Abort erwartet wird. |
| Wait-Node Condition-Modus (Polling) fehlt im UI-Pfad; nur Duration und Event verfügbar. | Externes Blueprint-Logik für den Condition-Check nutzen, dann `FireEvent` aufrufen. |

### Variablen

| Problem | Empfohlener Workaround |
| --- | --- |
| SetVariable-Node schreibt aktuell nur Dialogue-Scope; kein direkter Schreibpfad auf Participant-Scope. | Blueprint-SideEffect nutzen, der `Participant → SetPersistentVariable` direkt aufruft. |
| UI-Pfad für Tag-Typ-Variablen fehlt. | String-Wert mit GameplayTag-Pfad als Fallback. |

### Audio

| Problem | Empfohlener Workaround |
| --- | --- |
| Node-Level 2D-Override (`bOverride2D`) fehlt auf SayLine und PlaySound. | Speaker-Level `AudioModeOverride = Force2D` setzen. |
| `VolumeMultiplier` / `PitchMultiplier` auf SayLine fehlen; nur auf PlaySound verfügbar. | Auf Speaker-Ebene setzen. |
| Babel halb-funktional: Sound-Quality, Konfigurierbarkeit, Per-Speaker-Depth in Arbeit. | Eigene `BlipSounds` zuweisen statt prozeduralen Modus nutzen. |
| Babel Global Off: Flag vorhanden, vollständige Stummschaltung ohne Voice-Asset noch nicht final verifiziert. | Im eigenen Projekt testen, ob Stille erreicht wird. |

### Kamera

| Problem | Empfohlener Workaround |
| --- | --- |
| CameraFocus FOV-Restore läuft im Instance-Cleanup global; Original-FOV wird nicht pro Node gespeichert. | Nur ein einziges FOV-Override pro Dialog nutzen. |

### Input

| Problem | Empfohlener Workaround |
| --- | --- |
| `RestoreGameInputMode()` setzt hart auf `GameOnly`; der vorherige Modus wird nicht gecacht. | Nach `OnDialogueEnded` den eigenen Input-Modus manuell setzen. |

### Editor

| Problem | Empfohlener Workaround |
| --- | --- |
| Zyklus-Prüfung: Compiler erkennt Zyklen, das Schema lehnt sie aber beim Verbinden noch nicht ab. | Manuell aufpassen beim Verdrahten von Knot-Chains. |
| Live-Validation deaktiviert wegen Re-Entry-Problem. | Compile-Button nach jeder Änderung drücken. |
| Minimap-Widget existiert, ist keinem Tab zugeordnet. | Outline-Panel nutzen. |

### API & Bridge

| Problem | Empfohlener Workaround |
| --- | --- |
| Subscription-Hook für externe Systeme auf Choice-Tags nicht vollständig. | `OnChoiceMade`-Delegate abonnieren, ChoiceTags manuell aus `GetPendingChoices` nachschlagen. |
| Bridge-Write-Methoden (`SelectChoice`, `ForceAdvance`, `SetVariable`) exposed, aber nicht vollständig getestet. | Eigene Integrations-Tests vor Release schreiben, wenn Bridge kritisch genutzt wird. |
| QuickSave-Helper: API-Vertrag definiert, Implementierung wird finalisiert. | Projekt-eigenes SaveGame mit `ArIsSaveGame` nutzen. |

---

## Erledigte Issues (Auswahl)

Diese Punkte galten früher als offen und sind inzwischen behoben:

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
- Rich-Text-Decorators `<color>` und `<b>` funktional.
- Komponenten-basierte UMG-Widget-Variante mitgeliefert.

---

## Neues Issue melden

Nutze den GitHub-Issues-Tracker oder melde dich im Support-Kanal. Bitte immer mitliefern:

1. **Repro-Steps** — Was klicken, was erwarten, was passiert.
2. **Log-Auszug** — Nur die relevanten Zeilen aus `Saved/Logs/`.
3. **Plugin-Version** — Sichtbar in `MayDialogue.uplugin` unter `VersionName`.
