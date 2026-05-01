# Node Reference

Here you will find all Node types that ship with MayDialogue. Each Node page explains its purpose, properties, runtime behavior, and typical use cases.

## Categories

- [Core Nodes](core/README.md) — Structure and flow control (Entry, Exit, SayLine, PlayerChoice, Branch …).
- [Action Nodes](actions/README.md) — Standalone actions in the graph (CameraFocus, ApplyEffect, SetVariable …).
- [Sub-Nodes](sub-nodes/README.md) — Composition building blocks that live as pills inside parent Nodes (Requirement, Choice, SideEffect).

## Shared Base

Every Node inherits from `UMayDialogueNode_Base`. This gives three properties available on **every** Node:

| Field | Type | Meaning |
| --- | --- | --- |
| `Requirements` | Array | Conditions that must be met for the Node to be entered. |
| `SideEffects` | Array | Inline actions executed when the Node is entered. |
| `FailBehavior` | Enum | What happens when Requirements fail: `Skip` (skip the Node) or `Abort` (abort the dialogue). |
| `EditorComment` | FText | A note for yourself in the graph — has no runtime behavior. |

> 📸 **Image placeholder:** `node-base-details.png` — Details panel of any Node.
> *Setup:* Select a SayLine or Branch Node. The Details panel should show: section `Node|Requirements` (array empty), `Node|SideEffects` (array empty), `FailBehavior = Skip`, `EditorComment` (empty). Demonstrates the shared base of all Nodes.

## Blueprint Extensibility

All Node base classes are `Blueprintable`. Derive a Blueprint subclass, implement `ExecuteNode` as an Event — the new Node automatically appears in the editor's context menu.

Details: [Extension → Custom Nodes](../extension/custom-nodes.md).

## Quick Overview

### Core

| Node | Purpose |
| --- | --- |
| [Entry](core/entry.md) | Start point. One per asset. |
| [Exit](core/exit.md) | End point with status (Completed / Failed). |
| [Say Line](core/say-line.md) | A line spoken by a Speaker. |
| [Player Choice](core/player-choice.md) | Player selects from options. |
| [Branch](core/branch.md) | Automatic conditional branching. |
| [Random Line](core/random-line.md) | Random output from multiple variants. |
| [Wait](core/wait.md) | Pauses on time, event, or condition. |
| [Link](core/link.md) | Jumps to another dialogue asset. |
| [SubGraph](core/sub-graph.md) | Expands an internal sub-graph. |

### Actions

| Node | Purpose |
| --- | --- |
| [Camera Focus](actions/camera-focus.md) | Camera blend to a Speaker. |
| [Camera Shake](actions/camera-shake.md) | Trigger a camera shake. |
| [Play Animation](actions/play-animation.md) | Montage on a Participant. |
| [Apply Effect](actions/apply-effect.md) | Apply a GAS GameplayEffect. |
| [Set Variable](actions/set-variable.md) | Set a dialogue variable. |
| [Fire Event](actions/fire-event.md) | Fire a GameplayTag event. |
| [Play Sound](actions/play-sound.md) | Play a non-voice sound. |
| [Add Tag](actions/add-tag.md) | Set a loose tag. |
| [Remove Tag](actions/remove-tag.md) | Remove a loose tag. |
| [Trigger Cue](actions/trigger-cue.md) | GameplayCue one-shot. |

### Sub-Nodes

| Sub-Node | Purpose |
| --- | --- |
| [Requirement](sub-nodes/requirement.md) | Condition check with three results. |
| [Choice](sub-nodes/choice.md) | Answer option on a PlayerChoice. |
| [SideEffect](sub-nodes/side-effect.md) | Inline action on the parent Node. |
