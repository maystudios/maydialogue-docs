---
description: What you can extend — Nodes, Requirements, SideEffects, Bridge.
---

# Extension

MayDialogue is designed for you to bring project-specific logic directly into the plugin ecosystem. Everything you extend appears automatically in the editor pickers — no registration step, no boilerplate.

## What Is Extensible?

| Extension Point | How | Purpose |
| --- | --- | --- |
| **Custom Node Types** | Blueprint subclass of `UMayDialogueNode_Base` | Project-specific graph steps (e.g. "Notify Quest System") |
| **Custom Requirements** | Blueprint subclass of `UMayDialogueRequirement` | Project-specific conditions (e.g. "Quest completed", "Item in inventory") |
| **Custom SideEffects** | Blueprint subclass of `UMayDialogueSideEffect` | Project-specific actions (e.g. "Unlock achievement", "Quest progress +1") |
| **Bridge Interface** | Blueprint **or** C++ implementation of `IMayDialogueBridge` | Route dialogue events to external systems, read/write variables |
| **Bridge Event Binding** | Subscribe to Subsystem delegates | Global listener for dialogue start, end, abort |

## How Discovery Works

UE's reflection system scans all `UClass` subclasses at runtime. As soon as a Blueprint or C++ class is compiled, it automatically appears:

* In the **Requirement pickers** of every Choice, Branch, and SayLine.
* In the **SideEffect arrays** of every node.
* In the **context menu** of the dialogue graph (for custom nodes).
* In the **interface picker** when you create a Blueprint class that implements `IMayDialogueBridge` — the Subsystem detects your implementation automatically.

You register nothing manually.

> 📸 **Image placeholder:** `ext-overview-picker.png` — "Add Requirement" dropdown with a custom Blueprint class in the list.
> *Setup:* MayDialogue editor open, Choice node selected. In the Details panel: "Add Requirement" dropdown expanded. List shows standard classes (`Has Gameplay Tag`, `Check GAS Attribute`, …) and below them the custom Blueprint class `BP_Req_QuestCompleted` (under category "Quest"). The latter is highlighted.

## Blueprint-First Principle

All base classes are `Blueprintable` and `EditInlineNew`. This means:

* **Blueprint is sufficient for 90% of all extensions.** You click, not type.
* C++ makes sense when you need deep integration with your own C++ systems or have performance-critical logic.

Each page in this section shows the Blueprint approach first, followed by a C++ variant. The [C++ Fundamentals](cpp-fundamentals.md) (Build.cs setup, BlueprintNativeEvent pattern, module reload) are documented once and apply equally to all three extension types.

## Chapters in Detail

* [Custom Nodes](custom-nodes.md) — New graph steps with custom behavior.
* [Custom Requirements](custom-requirements.md) — New conditions for Choices, Branches, and SayLines.
* [Custom SideEffects](custom-side-effects.md) — New inline actions on any node.
* [Bridge Implementation](bridge-implementation.md) — Route dialogue events to external systems.
* [C++ Extension — Fundamentals](cpp-fundamentals.md) — Build.cs, BlueprintNativeEvent, module reload, live coding caveats.

> 📸 **Image placeholder:** `ext-overview-architecture.png` — Diagram: base class hierarchy with Blueprint subclasses as extension points.
> *Setup:* Simple UML-like diagram. Four base class boxes: `UMayDialogueNode_Base`, `UMayDialogueRequirement`, `UMayDialogueSideEffect`, `IMayDialogueBridge`. Below each, example subclasses as dashed boxes: `BP_DN_NotifyQuest`, `BP_Req_QuestCompleted`, `BP_SE_QuestProgress`, `BP_QuestBridge` (Blueprint). All four base classes have the label "Blueprintable". Arrows from subclasses to base classes.

## Best Practices — Quick Reference

* **One class, one responsibility.** `BP_Req_QuestActive` checks quest status — and nothing else.
* **Set display names.** Enter a meaningful name in Class Settings — it appears in pickers and pill labels.
* **Add null guards.** Check whether your subsystem is present. Return `Passed` (Requirement) or do nothing (SideEffect) when the subsystem is missing — never crash.
* **Respect fail modes.** `bHideOnFail` in Requirements should remain designer-configurable.

> 📸 **Image placeholder:** `ext-overview-pill-labels.png` — Graph view with custom SideEffect pills on a SayLine, readable display names.
> *Setup:* MayDialogue editor, SayLine node expanded. SideEffects array shows three pills: "Quest Progress +1 (KillDragon)", "Grant Achievement: DragonSlayer", "Add Tag: Story.DragonDefeated". Pill labels come from the `GetDisplayDescription` return value of the respective classes.
