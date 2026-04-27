# Sub-Nodes

Sub-Nodes sind Kompositions-Bausteine, die innerhalb eines Eltern-Nodes leben — nicht als eigene Box im Graph, sondern als Pill im Node-Body. Du siehst und bearbeitest sie direkt dort, ohne in ein separates Asset wechseln zu müssen.

## Die drei Sub-Node-Typen

| Sub-Node | Zweck | Eltern-Nodes |
| --- | --- | --- |
| [Requirement](requirement.md) | Bedingungscheck mit drei Ergebnissen | Alle Nodes, Branch (Condition), Choice |
| [Choice](choice.md) | Antwort-Option auf einem PlayerChoice-Node | PlayerChoice |
| [SideEffect](side-effect.md) | Inline-Aktion beim Betreten des Eltern-Nodes | Alle Nodes, Choice |

## Gemeinsames Konzept

Alle drei Sub-Node-Typen sind `EditInlineNew`, `BlueprintType` und `Blueprintable`:

- **`EditInlineNew`**: Du erstellst sie direkt im Details-Panel — kein separates Asset-Erstellen.
- **`Blueprintable`**: Du kannst eigene Subklassen in Blueprint anlegen. Sie erscheinen sofort in der Auswahl-Liste.

> 📸 **Bild-Platzhalter:** `subnodes-overview-pills.png` — Node-Body mit mehreren Sub-Node-Pills.
> *Setup:* SayLine-Node auswählen. Im Node-Body sichtbar: eine Requirement-Pill (`HasTag Story.Met.Guard`) und zwei SideEffect-Pills (`AddTag Story.Greeted`, `TriggerCue GameplayCue.UI.Flash`). Details-Panel zeigt die Arrays daneben.

## Erweiterbarkeit

{% hint style="success" %}
**Requirement und SideEffect sind Blueprint-subclassable.** Das ist das primäre Erweiterungs-Pattern von MayDialogue:

- Eigene Bedingungslogik → Blueprint-Subklasse von `UMayDialogueRequirement`.
- Eigene Inline-Aktionen → Blueprint-Subklasse von `UMayDialogueSideEffect`.

Details: [Extension → Custom Requirements](../../extension/custom-requirements.md) und [Extension → Custom SideEffects](../../extension/custom-side-effects.md).
{% endhint %}

## Hierarchie — Wo welcher Sub-Node sitzt

| Position | Sub-Node | Ausführungszeitpunkt |
| --- | --- | --- |
| Requirements auf **Node** | Requirement | Vor Node-Ausführung — bestimmt ob Node übersprungen wird. |
| Requirements auf **Branch (Condition)** | Requirement | Während Branch — bestimmt True/False-Pfad. |
| Requirements auf **Choice** | Requirement | Bei Choices-Anzeige und bei Auswahl — bestimmt Availability. |
| Choices auf **PlayerChoice** | Choice | Bei Choices-Anzeige. |
| SideEffects auf **Node** | SideEffect | Vor `ExecuteNode` — beim Betreten. |
| SideEffects auf **Choice** | SideEffect | Bei `SelectChoice` — wenn die Option gewählt wird. |
