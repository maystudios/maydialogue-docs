# Node-Referenz

Hier findest du alle Node-Typen, die MayDialogue mitliefert. Jede Node-Seite erklärt Zweck, Properties, Laufzeit-Verhalten und typische Einsatzszenarien.

## Kategorien

- [Core-Nodes](core/README.md) — Struktur und Fluss-Steuerung (Entry, Exit, SayLine, PlayerChoice, Branch …).
- [Action-Nodes](actions/README.md) — Eigenständige Aktionen im Graph (CameraFocus, ApplyEffect, SetVariable …).
- [Sub-Nodes](sub-nodes/README.md) — Kompositions-Bausteine die als Pills in Eltern-Nodes leben (Requirement, Choice, SideEffect).

## Gemeinsame Basis

Jeder Node erbt von `UMayDialogueNode_Base`. Daraus folgen drei Properties, die auf **jedem** Node verfügbar sind:

| Feld | Typ | Bedeutung |
| --- | --- | --- |
| `Requirements` | Array | Bedingungen, die erfüllt sein müssen, damit der Node betreten wird. |
| `SideEffects` | Array | Inline-Aktionen, die beim Betreten des Nodes ausgeführt werden. |
| `FailBehavior` | Enum | Was passiert, wenn Requirements fehlschlagen: `Skip` (Node überspringen) oder `Abort` (Dialog abbrechen). |
| `EditorComment` | FText | Notiz für dich im Graph — hat kein Laufzeit-Verhalten. |

> 📸 **Bild-Platzhalter:** `node-base-details.png` — Details-Panel eines beliebigen Nodes.
> *Setup:* Einen SayLine- oder Branch-Node auswählen. Im Details-Panel müssen sichtbar sein: Abschnitt `Node|Requirements` (Array leer), `Node|SideEffects` (Array leer), `FailBehavior = Skip`, `EditorComment` (leer). Zeigt damit die gemeinsame Basis aller Nodes.

## Blueprint-Erweiterbarkeit

Alle Node-Basisklassen sind `Blueprintable`. Leite eine Blueprint-Subklasse ab, implementiere `ExecuteNode` als Event — der neue Node erscheint automatisch im Kontext-Menü des Editors.

Details: [Extension → Eigene Nodes](../extension/custom-nodes.md).

## Kurz-Übersicht

### Core

| Node | Zweck |
| --- | --- |
| [Entry](core/entry.md) | Startpunkt. Einer pro Asset. |
| [Exit](core/exit.md) | Endpunkt mit Status (Completed / Failed). |
| [Say Line](core/say-line.md) | Eine Zeile, die ein Sprecher sagt. |
| [Player Choice](core/player-choice.md) | Spieler wählt aus Optionen. |
| [Branch](core/branch.md) | Automatische Verzweigung nach Bedingung. |
| [Random Line](core/random-line.md) | Zufällige Ausgabe aus mehreren Varianten. |
| [Wait](core/wait.md) | Pausiert auf Zeit, Event oder Bedingung. |
| [Link](core/link.md) | Springt in ein anderes Dialog-Asset. |
| [SubGraph](core/sub-graph.md) | Klappt einen internen Sub-Graphen auf. |

### Actions

| Node | Zweck |
| --- | --- |
| [Camera Focus](actions/camera-focus.md) | Kamera-Blend auf Sprecher. |
| [Camera Shake](actions/camera-shake.md) | Camera-Shake auslösen. |
| [Play Animation](actions/play-animation.md) | Montage auf Participant. |
| [Apply Effect](actions/apply-effect.md) | GAS-GameplayEffect anwenden. |
| [Set Variable](actions/set-variable.md) | Dialog-Variable setzen. |
| [Fire Event](actions/fire-event.md) | GameplayTag-Event feuern. |
| [Play Sound](actions/play-sound.md) | Nicht-Voice-Sound abspielen. |
| [Add Tag](actions/add-tag.md) | Loose-Tag setzen. |
| [Remove Tag](actions/remove-tag.md) | Loose-Tag entfernen. |
| [Trigger Cue](actions/trigger-cue.md) | GameplayCue one-shot. |

### Sub-Nodes

| Sub-Node | Zweck |
| --- | --- |
| [Requirement](sub-nodes/requirement.md) | Bedingungscheck mit drei Ergebnissen. |
| [Choice](sub-nodes/choice.md) | Antwort-Option auf PlayerChoice. |
| [SideEffect](sub-nodes/side-effect.md) | Inline-Aktion am Eltern-Node. |
