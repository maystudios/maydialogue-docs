# Bekannte Issues

Stand: **2026-04-20** — Plugin-Version 0.1.0 (Beta).

Diese Liste spiegelt die `BACKLOG.md` im Plugin-Root. Aktualisierungen erscheinen dort zuerst.

## Offen

| # | Bereich | Problem | Empfohlener Workaround |
| --- | --- | --- | --- |
| 1 | Widget / Viewport | Static-Widget überlebt Level-Teardown; UI bindet sich nach Level-Wechsel nicht neu. | `Subsystem->StopAllDialogues()` beim Level-Wechsel aufrufen, oder Widget manuell aus dem Viewport entfernen. |
| 2 | Wait-Node | Wait-Timer überlebt Dialog-Abort unter bestimmten Cleanup-Pfaden. | Halte Wait-Dauer kurz in risikoreichen Pfaden; oder feuere ein Timeout-Event, um frühzeitig aufzuräumen. |
| 3 | PlayAnimation-Node | Montage-End-Delegates bleiben bei Abort gebunden. | In risikoreichen Pfaden `bWaitForMontageEnd = false` setzen; oder Animation-End manuell verdrahten. |
| 4 | Input-Mode-Restore | `RestoreGameInputMode()` setzt hart auf `GameOnly`, cached nicht den vorherigen Modus. | Nach `OnDialogueEnded` eigenen Input-Modus setzen. |
| 5 | Node-Fail-Behavior | `EMayDialogueNodeFailBehavior` Feld existiert, wird aber in `ExecuteNode_Implementation` noch nicht konsequent geprüft. | Requirements auf Nodes bewusst redundant auf Outputs legen, falls Abort erwartet wird. |
| 6 | Knot-Zyklus | Compiler prüft Zyklen, Schema lehnt sie aber nicht beim Verbinden ab. | Manuell aufpassen beim Drahten. |
| 7 | Live-Validation | Aktuell deaktiviert wegen Re-Entry-Problem. | Compile-Button nach Änderungen drücken. |
| 8 | Minimap | Widget existiert, ist keinem Tab zugeordnet. | Outline-Panel nutzen. |
| 9 | Rich-Text-Decorator | `<color>` + `<b>` waren früher nicht funktional; **jetzt funktional** in aktueller Version. | – (erledigt) |
| 10 | UMG-Zerlegung | Primär-Widget historisch monolithisch; komponenten-basierte Version liefert die Plugin-Version jetzt mit. | – |
| 11 | UMG-Themes | Horror/VN/RPG Starter-Kits noch nicht mitgeliefert. | Eigenes Theme aufsetzen (siehe [Themes](../ui/themes.md)). |
| 12 | Wait-Condition-Modus | Duration + Event vorhanden, polling Condition-Modus fehlt im UI-Pfad. | Externes Blueprint-Logik für Condition-Check nutzen, dann FireEvent. |
| 13 | SetVariable Participant-Scope | Aktueller Node schreibt nur Instance-Scope. | Blueprint-SideEffect, der direkt `Part->SetPersistentXxx` aufruft. |
| 14 | SetVariable Tag-Typ | UI-Pfad für Tag-Variablen fehlt. | String-Value mit GameplayTag-Pfad als Fallback. |
| 15 | `bOverride2D` auf SayLine/PlaySound | Node-Level 2D-Override fehlt. | Speaker-Level `AudioModeOverride = Force2D` setzen. |
| 16 | SayLine `VolumeMultiplier` / `PitchMultiplier` | Nur auf PlaySound vorhanden, auf SayLine fehlt. | Speaker-Level setzen. |
| 17 | CameraFocus FOV-Restore per Node | Restore läuft im Instance-Cleanup global; Node-seitig wird Original-FOV nicht gehalten. | Ein einziges FOV-Override pro Dialog nutzen. |
| 18 | Dialogue-Events → GameplayCues | Manuelle TriggerCue-Nodes erforderlich. | So wie dokumentiert nutzen; automatisches Mapping nicht geplant. |
| 19 | Choice-Tag-Binding extern | Subscription-Hook für externe Systeme auf Choice-Tags nicht vollständig. | `OnChoiceMade`-Delegate abonnieren, ChoiceTags manuell aus `GetPendingChoices` nachschlagen. |
| 20 | Bridge-API | Einzelne Write-Methoden (`SelectChoice`, `ForceAdvance`, `SetVariable`) sind exposed, aber nicht voll durchgetestet. | Bei kritischer Nutzung: eigene Integrations-Tests vor Release. |
| 21 | Babel-Qualität | Aktuell halb-funktional; Sound-Quality, Konfigurierbarkeit, Per-Speaker-Depth sind in Arbeit. | Eigene `BlipSounds` zuweisen statt Procedural. |
| 22 | Babel Global Off | Flag vorhanden; vollständige Silence ohne Voice noch nicht final verifiziert. | Test in konkretem Projekt durchführen. |
| 23 | QuickSave-Helper | API-Vertrag definiert, Impl wird finalisiert. | Projekt-eigenes SaveGame mit `ArIsSaveGame` nutzen. |

## Erledigt (kleine Auswahl aus dem BACKLOG)

* ApplyEffect als Action-Node implementiert.
* Debugger Step-Into / Step-Out implementiert.
* Participant `DefaultDialogue` + `SetActiveDialogue()` / `GetActiveDialogue()` / `StartDefaultDialogue()`.
* Participant `AttenuationOverride` (AdvancedDisplay).
* 3-Level Audio Settings (Plugin → Speaker → Node).
* Audio-Playback im Editor-Preview mit Culture-Selection.
* PreviewRunner / FindResults als eigene Dateien.
* Auto-Layout-Button (Sugiyama).
* Mini-Graph Knot-Cycle-Detection im Slate-Widget.
* Compiler-ResolveKnotChain VisitedSet.
* Compiler Choice-Target GUID-Invalidierung.
* Instance unbounded immediate transitions gefixt (MaxHops + VisitedNodes).
* Random-Line State per-Instance (nicht mehr auf Asset-Node).
* Choice-Timeout-State bei neuem PresentChoices gecleart.
* Subsystem-Deinitialize ruft CleanupCompletedDialogues.
* Validator prüft async Nodes ohne Continuation.
* Failed Dialogue-Start: Entry-Point-Check vor Instance-Erzeugung.

## Neue Issues melden

Einen neuen Bug / Wunsch anlegen: via Perforce-interne Tracker-Konvention deines Teams, oder direkt ins `BACKLOG.md`.

Bitte immer:

* **Repro-Steps** (was klicken, was erwarten, was passiert).
* **Log-Auszug** (nur relevante Zeilen).
* **Asset-Version** + **Plugin-Version**.
