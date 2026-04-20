# Requirement

Ein **Requirement** ist eine Bedingungs-Prüfung, die ein `EMayDialogueRequirementResult` liefert.

## Die drei Ergebnisse

```cpp
enum class EMayDialogueRequirementResult
{
    Passed,             // Alles gut
    FailedButVisible,   // Zeigen, aber grau/disabled
    FailedAndHidden,    // Gar nicht anzeigen
};
```

Dieses Drei-Stufen-Modell ermöglicht das klassische RPG-Muster *„Siehe-Option-nur-wenn-du-sie-hast"* vs. *„Sehe-sie-immer-aber-erst-mit-Bedingung-wählbar"* je nach Situation – der Designer wählt pro Instanz.

## Basis-Klasse

`UMayDialogueRequirement` (Abstract, EditInlineNew, BlueprintType, Blueprintable):

| Property | Typ | Zweck |
| --- | --- | --- |
| `Description` | `FText` | Tooltip / Editor-Display. |
| `bHideOnFail` | `bool` | Wenn der Check fehlschlägt: Hidden (`true`) oder Visible-aber-grau (`false`). |

Virtuelle Methoden:

```cpp
virtual EMayDialogueRequirementResult IsRequirementSatisfied(const FMayDialogueContext&) const;
virtual FText GetDisplayDescription() const;
```

## Standard-Requirements (im MayDialogueGAS-Modul)

| Klasse | Prüft | Wichtige Properties |
| --- | --- | --- |
| `UMayDlgRequirement_HasTag` | ASC trägt einen bestimmten Tag | `RequiredTag`, `bCheckOnInstigator` |
| `UMayDlgRequirement_CheckAttribute` | Attribut-Vergleich | `Attribute`, `ComparisonOp` (>/</==/>=/<=/!=), `ComparisonValue`, `bCheckOnInstigator`, `FailureResult` |
| `UMayDlgRequirement_HasAbility` | ASC hat eine bestimmte Ability-Klasse | `RequiredAbility`, `bCheckOnInstigator` |

Details siehe [GAS → Requirements](../../gas/requirements.md).

## Komposition: Mehrere Requirements

Sub-Nodes werden als **Array** am Eltern-Node abgelegt. `UMayDialogueRequirement::EvaluateAll(Requirements, Context)` kombiniert sie:

* Alle Passed → Passed.
* Irgendeiner FailedAndHidden → FailedAndHidden.
* Sonst irgendeiner FailedButVisible → FailedButVisible.

(FailedAndHidden > FailedButVisible > Passed im Merge-Operator.)

## Wo einsetzen?

```
[Branch]
  Condition: HasTag("Story.Met.Guard")
    ↳ True-Pfad / False-Pfad

[PlayerChoice]
  Choice 0: "Freund" (keine Requirements)
  Choice 1: "Passwort" (Requirement: HasTag "Story.HeardPassword", bHideOnFail=true)
  Choice 2: "König"    (Requirement: CheckAttribute Reputation >= 50, FailureResult=FailedButVisible)
```

## Eigene Requirements

Per Blueprint:

1. Rechtsklick im Content Browser → Blueprint Class → `UMayDialogueRequirement` als Parent wählen.
2. Event `IsRequirementSatisfied` überschreiben.
3. Blueprint kompilieren.
4. Im Node-Details-Panel → Add Requirement → deine neue Klasse erscheint in der Liste.

Siehe [Extension → Custom Requirements](../../extension/custom-requirements.md).

## Anmerkungen

* **FailBehavior** (auf Node-Ebene) entscheidet, was passiert, wenn Requirements auf einem *Node* (nicht Choice) fehlschlagen: Skip oder Abort. Aktuell noch als Backlog-Item 5 nur teilweise verdrahtet.
