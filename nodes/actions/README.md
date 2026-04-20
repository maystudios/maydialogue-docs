# Action-Nodes

Action-Nodes sind **prominente Aktionen** im Graph – jede als eigene Box sichtbar, jede deaktivierbar per Breakpoint, jede kommentierbar. Sie sind die Antwort auf: *„Diese Aktion ist der Hauptpunkt dieses Schrittes."*

Wichtig: **Dieselben Aktionstypen existieren auch als [SideEffect-Sub-Nodes](../sub-nodes/side-effect.md)**. Designer wählt pro Situation:

* Hauptschritt? → **Action-Node** (eigene Box).
* Nebenschritt? → **SideEffect-Sub-Node** (Pill im Body).

| Node | Kategorie | Zweck |
| --- | --- | --- |
| [Camera Focus](camera-focus.md) | Kamera | Kamera-Blend auf Sprecher. |
| [Camera Shake](camera-shake.md) | Kamera | Camera-Shake. |
| [Play Animation](play-animation.md) | Animation | Montage auf Participant. |
| [Apply Effect](apply-effect.md) | GAS | GameplayEffect anwenden. |
| [Set Variable](set-variable.md) | Daten | Variable setzen. |
| [Fire Event](fire-event.md) | Daten | GameplayTag-Event feuern. |
| [Play Sound](play-sound.md) | Audio | Non-Voice-Sound. |
| [Add Tag](add-tag.md) | GAS | LooseGameplayTag setzen. |
| [Remove Tag](remove-tag.md) | GAS | LooseGameplayTag entfernen. |
| [Trigger Cue](trigger-cue.md) | GAS | GameplayCue one-shot. |

## Gemeinsames Verhalten

Alle Action-Nodes:

* Haben einen Input- und einen Output-Pin.
* Sind nach Ausführung **Immediate Advance** (kein Warten auf Spieler).
* Ausnahme: `PlayAnimation` mit `bWaitForMontageEnd=true` ist async.
* Werden im Debugger per Breakpoint pausierbar.
