---
description: Quick lookup — properties, signatures, enums, structs.
---

# Reference

The Reference section is a lookup resource. No explanations, no narratives — just the bare facts: property tables, function signatures, enum values.

## What You'll Find Here

| Page | Contents | When to use |
|---|---|---|
| [Project Settings](project-settings.md) | All fields of `UMayDialogueSettings` with type, default, and meaning. | Forgot a setting name or unclear about the default value. |
| [Editor Settings](editor-settings.md) | All fields of `UMayDialogueEditorSettings` — node colors, debug highlights. | Customize node colors, define team defaults. |
| [API: Library](api-library.md) | All methods of `UMayDialogueLibrary` as a table with parameters and return values. | Blueprint quick-start, looking up a signature. |
| [API: Subsystem](api-subsystem.md) | Public methods and delegates of `UMayDialogueSubsystem`. | C++ integration, event binding, lifecycle control. |
| [API: Delegates](api-delegates.md) | All delegate types with signature and fire time. | Hooking a game system into dialogue phases. |
| [Types & Enums](types.md) | All enums and important structs — values and one-sentence explanations. | Resolving autocomplete confusion, looking up struct fields. |

## What Is Not Here

- **Concepts & Architecture** → [Core Concepts](../concepts/README.md)
- **Node documentation** → [Node Reference](../nodes/README.md)
- **GAS-specific Requirements/Actions** → [GAS Integration](../gas/README.md)
- **UI widget APIs** → [UI System](../ui/README.md)

## How to Read the Property Tables

The same schema everywhere:

| Property | Type | Default | Meaning |
|---|---|---|---|
| `PropertyName` | `TType` | `Value` | One sentence describing what the setting controls. |

Type conventions:

- `TSoftObjectPtr<T>` / `TSoftClassPtr<T>` — Lazy reference, loaded only on the first dialogue start.
- `FGameplayTag` / `FGameplayTagContainer` — Tags from the UE GameplayTags system.
- No default given — zero-initialized or `nullptr`.

## Which Class Is Responsible for What?

```text
UMayDialogueLibrary          (Blueprint helper, stateless)
    │ delegates to
UMayDialogueSubsystem        (Orchestrator, one instance per world)
    │ creates and manages
UMayDialogueInstance         (running conversation, has all delegates)
    │ reads from
UMayDialogueAsset            (the dialogue graph, immutable blueprint)

UMayDialogueParticipant      (actor component, binds actor to Instance)
    │ sits on
AActor (player, NPC, ...)

IMayDialogueBridge           (interface, implemented by the Subsystem)
```

> 📸 **Image placeholder:** `reference-class-overview.png` — Class diagram as an annotation diagram.
> *Setup:* No editor needed — hand-drawn or whiteboard photo of the diagram above. Class boxes labeled with class name + short description. Arrows show "delegates to", "creates", "sits on", "implements". Colors: Library=Blue, Subsystem=Orange, Instance=Green, Asset=Grey, Participant=Purple, Bridge=Dashed border.
