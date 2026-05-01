# Sub-Nodes

Sub-Nodes are composition building blocks that live inside a parent Node — not as their own box in the graph, but as a pill in the Node body. You see and edit them directly there, without switching to a separate asset.

## The Three Sub-Node Types

| Sub-Node | Purpose | Parent Nodes |
| --- | --- | --- |
| [Requirement](requirement.md) | Condition check with three results | All Nodes, Branch (Condition), Choice |
| [Choice](choice.md) | Answer option on a PlayerChoice Node | PlayerChoice |
| [SideEffect](side-effect.md) | Inline action when the parent Node is entered | All Nodes, Choice |

## Shared Concept

All three Sub-Node types are `EditInlineNew`, `BlueprintType`, and `Blueprintable`:

- **`EditInlineNew`**: You create them directly in the Details panel — no separate asset creation.
- **`Blueprintable`**: You can create your own subclasses in Blueprint. They immediately appear in the selection list.

> 📸 **Image placeholder:** `subnodes-overview-pills.png` — Node body with multiple Sub-Node pills.
> *Setup:* Select a SayLine Node. Visible in the Node body: a Requirement pill (`HasTag Story.Met.Guard`) and two SideEffect pills (`AddTag Story.Greeted`, `TriggerCue GameplayCue.UI.Flash`). Details panel shows the arrays alongside.

## Extensibility

{% hint style="success" %}
**Requirement and SideEffect are Blueprint-subclassable.** This is the primary extension pattern of MayDialogue:

- Custom condition logic → Blueprint subclass of `UMayDialogueRequirement`.
- Custom inline actions → Blueprint subclass of `UMayDialogueSideEffect`.

Details: [Extension → Custom Requirements](../../extension/custom-requirements.md) and [Extension → Custom SideEffects](../../extension/custom-side-effects.md).
{% endhint %}

## Hierarchy — Which Sub-Node Goes Where

| Position | Sub-Node | Execution Time |
| --- | --- | --- |
| Requirements on **Node** | Requirement | Before Node execution — determines whether the Node is skipped. |
| Requirements on **Branch (Condition)** | Requirement | During Branch — determines the True/False path. |
| Requirements on **Choice** | Requirement | When Choices are displayed and on selection — determines availability. |
| Choices on **PlayerChoice** | Choice | When Choices are displayed. |
| SideEffects on **Node** | SideEffect | Before `ExecuteNode` — when entering. |
| SideEffects on **Choice** | SideEffect | On `SelectChoice` — when the option is chosen. |
