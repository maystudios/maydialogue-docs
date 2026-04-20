# Exit

Der **Exit-Node** beendet den Dialog. Ein Asset kann mehrere Exits haben (z.B. *„Completed"* und *„Failed"* als separate Nodes).

## Runtime-Verhalten

`ExecuteNode` ruft `SideEffects` auf, dann:

```cpp
return FMayDialogueTaskResult::Abort();  // mit ExitStatus
```

Das Subsystem macht anschließend Cleanup (Audio stoppen, Camera restoren, UI schließen, Delegates feuern).

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `ExitStatus` | `EMayDialogueExitStatus` | `Completed` oder `Failed`. Wird in `OnDialogueEnded` durchgereicht. |
| `SideEffects` | Array | Letzte Aktionen vor dem Ende. |

## Pins

* **Input** – Eingehender Draht. Mehrere Eingänge sind erlaubt (Knot-Summen-Pattern).

## Exit-Status-Unterscheidung

* **Completed** – Dialog sauber beendet, positives Ende. Quest-System kann Haken setzen.
* **Failed** – Dialog vorzeitig oder schlecht beendet. Quest-System kann Alternative triggern.

Externe Hörer lesen den Status aus `OnDialogueEnded(Asset, ExitStatus, Duration, Instigator, Target)`.

## Typisches Pattern

```
... → [PlayerChoice] ── Output 0 ── [ApplyEffect] ── [SayLine] ── [Exit: Completed]
                      ── Output 1 ── [CameraShake] ── [SayLine] ── [Exit: Failed]
```

## Anmerkungen

* **Kein Exit-Node?** Validator warnt (aber nicht Error) – der Dialog kann theoretisch endlos loopen.
* Ein Exit beendet **den aktuellen Scope**. In einem verlinkten Sub-Dialog kehrt die Ausführung via Scope-Stack zum Caller zurück.
