---
description: MayDialogue and CommonConversation — inspiration, differences, and when to choose which.
---

# MayDialogue & CommonConversation

Epic Games ships an internal plugin called **CommonConversation** (CC) as part of Unreal Engine 5, used in the Lyra Starter Game and other Epic projects. MayDialogue has deliberately modeled its runtime architecture on CC — adopting the right ideas, filling in the gaps with a complete layer on top, and requiring no dependency on the CC plugin itself.

This chapter explains: What is CommonConversation? What did MayDialogue adopt from it, and why? What does MayDialogue add? And when should you consider CC over MayDialogue?

> 📸 **Image placeholder:** `cc-comparison-overview.png` — Side-by-side diagram: CC vs. MayDialogue.
> *Setup:* Two columns. Left "CommonConversation": boxes for Instance, Participant, Task-Result Loop, Sub-Node Composition — but no connecting lines to UI, Audio, or Editor. Right "MayDialogue": the same four core boxes (highlighted), plus connecting lines to Graph Editor, UI Layer (Slate + UMG), Audio Pipeline (3D + Babel), Camera Nodes, GAS Integration, Persistence, Debugger/Preview. Title: "Same foundation — complete build."

## What CommonConversation is

CommonConversation is Epic's internal dialogue runtime — a data model and runtime core for server-authoritative, replicable conversations. It is designed as a **raw material layer** used in Lyra and internal Epic projects: solid architectural foundations, intentionally without a finished UI, audio pipeline, or editor experience.

CC answers the question: "How do you structure a conversation as a server object with a clean replication pattern?" It does not answer: "How does the dialogue look for the player, how does it sound, and how do you build it in the editor?"

**CC is not an end-user product** — it is an architectural pattern Epic uses for its own projects. It is not available on the Marketplace and is not intended for direct use without a project-specific wrapper.

## Why there is no external CC dependency

**MayDialogue does not pull CommonConversation in as a dependency plugin.** The adopted patterns live in MayDialogue's own modules — as conceptual adoptions and adapted implementations. Anyone installing MayDialogue does not need to enable, know about, or configure CC.

Three reasons:

1. **CC is marked as an Epic-internal material layer** — no stability guarantees, no advance notice of changes.
2. **CC provides no UI/audio integration** — MayDialogue needs full control over its own pipeline.
3. **Simpler installation** — one plugin, one module set, no hidden dependencies.

Attribution is explicit: MayDialogue is architecturally inspired by CC, copies no Epic source code, and holds full rights over its own plugin implementation (see `CREDITS.md`).

## What MayDialogue adopted from CC

### Server-authoritative instance

CC's `UConversationInstance` is a plain `UObject` — not an actor, not a component — that encapsulates the running conversation and exists only on the server. MayDialogue mirrors this pattern 1:1 as `UMayDialogueInstance`.

The pattern is a deliberate choice: conversations contain game-relevant state transitions (variables, quest progress, choice selection). These must be server-authoritative so clients cannot spoof them. Clients see the conversation state through a client-RPC mirror; inputs travel through server RPCs.

### Participant component model

CC's `UConversationParticipantComponent` links actors in the level to entries in the conversation asset via a tag. MayDialogue adopts this model as `UMayDialogueParticipant` and extends it with portraits, persistent memory (`PersistentMemory`), camera offset anchors, audio overrides, and a `DefaultDialogue` reference.

### Declarative task-result nodes

CC's task-result pattern is elegant: nodes **describe** what they do and **return** how the dialogue should proceed — as an enum value (`FConversationTaskResult`). The traversal loop in the instance **interprets** the result. No node drives the state machine itself.

MayDialogue mirrors this pattern as `FMayDialogueTaskResult` with `EMayDialogueTaskResultType` (Advance / Pause+Choices / Abort, plus experimental return values). The central invariant remains the same: **nodes declare, the instance decides.**

### Sub-node composition

CC's TaskNode concept: a node carries Requirements (may I execute?), Choices (what can the player select?), and SideEffects (what happens on the side?) as embedded sub-objects. MayDialogue adopts this composition completely — requirements, choices, and side effects appear as compact pills in the node body, not as separate graph boxes.

### Scope stack for sub-conversations

CC's link/sub-conversation mechanic: a Link node switches to another conversation context, pushes a return point onto a stack, and pops it when the linked context ends. MayDialogue adopts this model for `Link` and `SubGraph` nodes — deep nesting without manual return-point code.

## What MayDialogue adds

### Visual graph editor

CC ships without its own asset editor. MayDialogue provides a complete custom EdGraph (`UMayDialogueGraph`): speaker colors in title bars, inline text, double-click inline editing, requirement/side-effect pills, a palette with drag-and-drop, an Outline panel, Find-in-Dialogue, auto-layout, SubGraph breadcrumbs, comment boxes, and reroute knots. **The graph reads like a screenplay** — this is the primary design goal of the editor.

### Finished UI in two layers

CC ships only `FClientConversationMessage` — a data payload. What happens with it is the project's responsibility.

MayDialogue ships two production-ready UI layers:

- **Slate debug widget** (`SMayDialogueWidget`): auto-spawned, no setup, works immediately — for prototypes and testing.
- **UMG component system** (`UMayDialogueWidget`): six individually swappable component widgets (DialogFrame, Speaker, Text, ChoiceList, ChoiceButton, SkipButton), a typewriter engine, rich-text decorators (`<shake>`, `<wave>`, `<color>`, `<b>`), and three included themes (Horror / Visual Novel / RPG).

### 3D audio pipeline and Babel

CC has no audio integration. MayDialogue ships:

- **Four-level fallback chain** (Plugin Settings → Speaker → Participant → Node) for spatial 3D and forced 2D playback.
- **Babel synthesized voices** (`UMayDialogueBabelSynth`): procedural placeholder voices for every NPC — in development you hear timing immediately without recording real VO. Shippable as a stylistic option (Fears-to-Fathom-style blip voices). Profiles are configurable as Data Assets per speaker.
- **VoicePerCulture map** per SayLine: multilingual VO with a BCP-47 fallback chain.

### Camera nodes

CC has no camera integration. MayDialogue ships:

- **CameraFocus** (three modes: SpeakerLook, CineCamera anchor by tag, LevelSequence with optional wait) — quick dialogue direction straight in the graph.
- **CameraShake** (`UCameraShakeBase`, scale, spatial falloff).
- Auto-focus speaker and auto-face partner as opt-ins on the Participant component.

### In-editor preview and PIE debugger

CC has no in-editor preview (PIE debugging only). MayDialogue ships:

- **Preview Runner**: a complete dialogue playthrough directly inside the asset editor — no need to start PIE. Typewriter, voice/Babel, choices, live variable edits all work.
- **PIE Debugger**: per-node breakpoints (persistent), step controls (over/into/out, including sub-graphs), active-node highlight, variable watch with write access.

### GAS integration as a first-class citizen

CC has no GAS bindings. MayDialogue ships fully integrated GAS requirements (HasTag, HasAbility, CheckAttribute), GAS actions/side effects (AddTag, RemoveTag, ApplyEffect, TriggerCue), and lifecycle cue bindings as a no-code GAS bridge — isolated in the `MayDialogueGAS` module but auto-loaded.

### Persistence and variables

CC has no built-in persistence. MayDialogue ships:

- **Dialogue-scope variables** (`FInstancedPropertyBag`): temporary variables that exist for the duration of a conversation.
- **Participant persistent memory** (`PersistentMemory`, SaveGame-flagged): variables that survive conversations and land in the SaveGame automatically.
- **QuickSave helper** (`UMayDialogueSaveHelper`) for projects without their own save system.

### 19 concrete nodes instead of abstract base classes

CC ships only abstract base classes (TaskNode, ChoiceNode, RequirementNode, SideEffectNode). MayDialogue ships 19 ready-to-use nodes (9 core nodes + 10 action nodes with full SideEffect parity) — and every base class is Blueprint- and C++-subclassable for your own node types.

## One deliberate difference: a single entry point per asset

CC allows multiple entry tags per conversation (Greeting, Farewell, Angry — a different tag is targeted depending on context). MayDialogue has **exactly one Entry node per asset**.

**Rationale**: clarity wins over flexibility. One asset, one clear entry point — the graph reads linearly. If you need different starting contexts, use separate assets or a Branch node right after the entry.

## Term translation table

If you're reading CC source code or Lyra documentation and looking for MayDialogue equivalents:

| CommonConversation | MayDialogue equivalent |
| --- | --- |
| `UConversationInstance` | `UMayDialogueInstance` |
| `UConversationParticipantComponent` | `UMayDialogueParticipant` |
| `UConversationRegistry` | *(not needed — assets are referenced directly)* |
| `FConversationTaskResult` | `FMayDialogueTaskResult` |
| `FClientConversationMessage` | `FMayDialogueMessage` |
| `FClientConversationOptionEntry` | `FMayDialogueChoiceEntry` |
| `UConversationNode` | `UMayDialogueNode_Base` |
| `UConversationRequirementNode` | `UMayDialogueRequirement` |
| `UConversationSideEffectNode` | `UMayDialogueSideEffect` |

## When to consider CommonConversation over MayDialogue

MayDialogue is the right choice for the vast majority of projects. There are, however, situations where staying CC-native makes more sense:

| Scenario | Recommendation |
| --- | --- |
| Working in a Lyra-based project with an existing CC stack and a dedicated programmer team | CC (no migration cost, stack already integrated) |
| Need multiple entry tags per conversation without separate assets | CC (multi-entry is a CC feature) |
| Building a multiplayer game with choice-voting mechanics for multiple simultaneous players | CC (choice voting and parallel conversations are CC core features) |
| Want a playable dialogue in 5 minutes — no boilerplate, no widget building, no audio setup | MayDialogue |
| Project needs GAS requirements and actions in dialogues | MayDialogue |
| Want finished UI themes, typewriter, Babel voices, and camera direction | MayDialogue |
| Solo developer or small team without a dedicated engine programmer | MayDialogue |
| Project is not a Lyra fork | MayDialogue (no shared base) |

{% hint style="info" %}
MayDialogue is not a CC wrapper and not a CC alternative — it is a standalone, complete dialogue system that uses the **same proven architectural patterns** and builds a complete product on top of the foundations CC deliberately leaves open.
{% endhint %}

## Summary

MayDialogue owes some of its most important architectural ideas to CommonConversation: the server-authoritative instance, the participant component model, the declarative task-result pattern, and sub-node composition. These foundations are solid, proven in Lyra projects, and deliberately not reinvented.

What MayDialogue builds on top — the visual editor, the UI layers, the audio pipeline, Babel voices, camera nodes, GAS integration, persistence, and the debugging tooling — is independent work that CC intentionally leaves open.

Next: [Graph & Visual Language](graph-visual-language.md).
