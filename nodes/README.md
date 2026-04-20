# Node-Referenz

Hier findest du **jeden Node-Typ**, den MayDialogue mitliefert, als eigene Seite. Die Nodes sind in drei Kategorien gegliedert:

* [Core-Nodes](core/README.md) – Struktur und Flow-Control (Entry, Exit, SayLine, PlayerChoice, Branch, …).
* [Action-Nodes](actions/README.md) – prominente Aktionen im Graph (CameraFocus, ApplyEffect, SetVariable, …).
* [Sub-Nodes](sub-nodes/README.md) – Kompositions-Bausteine (Requirement, Choice, SideEffect).

## Design-Prinzip

Alle Nodes leiten von **`UMayDialogueNode_Base`** ab und implementieren:

```cpp
virtual FMayDialogueTaskResult ExecuteNode_Implementation(const FMayDialogueContext& Context);
virtual void ExecuteClientEffects(const FMayDialogueContext& Context);
```

* `ExecuteNode` liefert ein [TaskResult](../concepts/instance-lifecycle.md#node-ausfuhrung) zurück, das der Instance sagt, wie weiter verfahren werden soll.
* `ExecuteClientEffects` ist nur für rein kosmetische Seiten-Effekte gedacht (Sounds, Particles), die nicht in die Gameplay-Logik eingreifen.

Zusätzlich hat jeder Node:

| Feld | Bedeutung |
| --- | --- |
| `NodeGuid` | Stabile Identität, vom Compiler vergeben. |
| `Requirements` | Array von `UMayDialogueRequirement`. Bei `FailedAndHidden` wird der Node gar nicht ausgeführt. |
| `SideEffects` | Array von `UMayDialogueSideEffect`. Werden vor dem eigentlichen `ExecuteNode` ausgeführt. |
| `FailBehavior` | `Skip` oder `Abort`, wenn Requirements fehlschlagen. |

## Blueprint-Erweiterbarkeit

**Alle Node-Basisklassen sind Blueprintable.** Wenn dir ein vordefinierter Node nicht reicht, leite eine Blueprint-Subklasse ab, implementiere `ExecuteNode` als Event-Graph – der Node erscheint automatisch im Kontext-Menü.

Details siehe [Extension → Eigene Nodes](../extension/custom-nodes.md).

## Kurz-Übersicht

### Core

| Node | Zweck |
| --- | --- |
| [Entry](core/entry.md) | Startpunkt. Einer pro Asset. |
| [Exit](core/exit.md) | Endpunkt (Completed / Failed). |
| [Say Line](core/say-line.md) | Eine Zeile, die ein Sprecher spricht. |
| [Player Choice](core/player-choice.md) | Spieler wählt aus Optionen. |
| [Branch](core/branch.md) | Automatische Verzweigung nach Bedingung. |
| [Random Line](core/random-line.md) | Zufällige Ausgabe-Option. |
| [Wait](core/wait.md) | Pausiert auf Duration / Event. |
| [Link](core/link.md) | Springt in anderes Asset. |
| [SubGraph](core/sub-graph.md) | Wechselt in internen Sub-Graph. |

### Actions

| Node | Zweck |
| --- | --- |
| [Camera Focus](actions/camera-focus.md) | Kamera-Blend auf Sprecher. |
| [Camera Shake](actions/camera-shake.md) | Camera-Shake auslösen. |
| [Play Animation](actions/play-animation.md) | Montage auf Participant. |
| [Apply Effect](actions/apply-effect.md) | GAS-GameplayEffect. |
| [Set Variable](actions/set-variable.md) | Variable setzen. |
| [Fire Event](actions/fire-event.md) | GameplayTag-Event feuern. |
| [Play Sound](actions/play-sound.md) | Non-Voice-Sound. |
| [Add Tag](actions/add-tag.md) | Loose-Tag setzen. |
| [Remove Tag](actions/remove-tag.md) | Loose-Tag entfernen. |
| [Trigger Cue](actions/trigger-cue.md) | GameplayCue one-shot. |

### Sub-Nodes

| Sub-Node | Zweck |
| --- | --- |
| [Requirement](sub-nodes/requirement.md) | Bedingungscheck mit drei Ergebnissen. |
| [Choice](sub-nodes/choice.md) | Antwort-Option auf PlayerChoice. |
| [SideEffect](sub-nodes/side-effect.md) | Inline-Aktion am Eltern-Node. |
