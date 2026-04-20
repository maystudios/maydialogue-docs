# Sub-Nodes

Sub-Nodes sind **kompositionelle Bausteine**, die innerhalb eines Eltern-Nodes leben – nicht als eigene Box im Graph, sondern als Pill im Body.

Drei Sub-Node-Typen:

* [Requirement](requirement.md) – Bedingungscheck mit drei Ergebnissen.
* [Choice](choice.md) – Antwort-Option auf einem PlayerChoice-Node.
* [SideEffect](side-effect.md) – Inline-Aktion beim Betreten eines Nodes.

## Gemeinsames Konzept

Alle drei sind `EditInlineNew`, BlueprintType, Blueprintable:

* `EditInlineNew` → sie werden im Details-Panel direkt erstellt, nicht als eigenes Asset.
* `Blueprintable` → eigene Subklassen in Blueprint möglich. Siehe [Extension](../../extension/custom-requirements.md).

Sub-Nodes leben **als Array auf dem Eltern-Node** und werden vom Eltern-Node in passenden Momenten abgefragt bzw. ausgeführt.

## Hierarchie

| Wo? | Zweck |
| --- | --- |
| Requirements auf **Branch** | Bestimmt, ob True / False / Fallback-Pfad genommen wird. |
| Requirements auf **Choice** | Bestimmt Availability (Passed / FailedButVisible / FailedAndHidden). |
| Requirements auf **Node** | Bestimmt, ob der Node überhaupt ausgeführt wird (oder übersprungen). |
| Choices auf **PlayerChoice** | Definieren die Antwort-Optionen. |
| SideEffects auf **jedem Node** | Inline-Aktionen beim Betreten. |
| SideEffects auf **Choice** | Werden bei SelectChoice ausgeführt. |
