---
description: The main building blocks of the plugin and how they work together — from a user perspective.
---

# Architecture Overview

MayDialogue consists of four main building blocks you should know as a user: the **dialogue asset**, the **participant component**, the **subsystem**, and the **UI layer**. This chapter shows how they connect — without going into internal class details.

> 📸 **Image placeholder:** `architecture-overview.png` — Overview diagram of all four main building blocks.
> *Setup:* A simple block diagram with four boxes: "Dialogue Asset (Disk)", "Subsystem (World)", "Participant Component (Actor)", "UI Widget". Arrows: Asset → Subsystem (loaded by), Participant Component → Subsystem (starts via), Subsystem → UI Widget (sends events to). No code, just boxes and labels.

## The four main building blocks

### 1. The dialogue asset

A dialogue asset (`UMayDialogueAsset`) is a file in your Content Browser. It contains:

- The **node graph** — the entire dialogue flow, visually as connected boxes.
- The **speaker list** — who speaks, with which portrait, color, and audio setup.
- The **variable declarations** — all variables that exist in the conversation.

The asset is pure **structure and data**. Nothing runs when you simply have it in the Content Browser. Only when a dialogue is started does the asset produce a running instance.

> 📸 **Image placeholder:** `asset-content-browser.png` — Content Browser with an open dialogue asset.
> *Setup:* Content Browser shows an asset `DA_Guard_Greeting` with the MayDialogue icon. Beside it the asset editor with an open graph, visible: Entry node (green), a SayLine (red title bar for the guard speaker), a PlayerChoice node with two choices, Exit node (red). Speakers panel upper right shows one entry "Guard" with an orange color chip.

### 2. The participant component

`UMayDialogueParticipant` is an **ActorComponent** you attach to every actor that appears in dialogues — NPCs, the player pawn, objects with a voice.

The component has two main responsibilities:

- **Identity**: It carries the `ParticipantTag` (e.g. `Dialogue.Speaker.Guard`). The plugin uses this at runtime to link the live actor in the level to the speaker entry in the asset.
- **Memory**: `PersistentMemory` stores variables that survive conversations (e.g. whether the player has already met the NPC).

> 📸 **Image placeholder:** `participant-component-details.png` — Details panel of the Participant component on an NPC.
> *Setup:* A guard actor selected in the level, Details panel shows the `MayDialogueParticipant` component. Visible fields: `ParticipantTag = Dialogue.Speaker.Guard`, `DisplayName = Guard`, `DefaultDialogue = DA_Guard_Greeting` (asset reference), `bAutoFacePartner = true`. Portrait slot is filled with a Texture2D reference.

### 3. The subsystem

`UMayDialogueSubsystem` is a **World Subsystem** — it exists once per world and lives automatically; you don't need to create it.

It is the only instance allowed to start and end new conversations. All three start paths (directly on the component, via the Blueprint library, or via code) ultimately converge at the subsystem.

The subsystem manages the active instance and cleans it up automatically after it ends.

> 📸 **Image placeholder:** `blueprint-start-dialogue.png` — Blueprint graph of an interaction trigger that starts a dialogue.
> *Setup:* A simple BP graph with: `Event BeginOverlap` → `Start Default Dialogue` (MayDialogueLibrary node). Node pins visible: `Instigator = Get Player Pawn`, `Target = Self (GuardActor)`. The start node is purple (library function), the output pin shows a connection to a `Print String` node with text "Dialogue started".

```cpp
// C++ — three equivalent ways to start a dialogue

// 1. On the component (OOP style)
Guard->FindComponentByClass<UMayDialogueParticipant>()->StartDefaultDialogue(Player);

// 2. Via the library (Blueprint-friendly)
UMayDialogueLibrary::StartDialogue(this, DA_Guard_Greeting, PlayerPawn, GuardActor);

// 3. Directly on the subsystem
UMayDialogueSubsystem::Get(this)->StartDialogue(DA_Guard_Greeting, PlayerPawn, GuardActor);
```

### 4. The UI layer

The plugin ships with a ready-made UI. You don't need to build a widget from scratch to display dialogues.

The UI layer listens to events from the active instance (`OnMessageReceived`, `OnChoicesPresented`) and renders text, portraits, choices, and typewriter effects. You can use the included widget directly, customize it, or replace it with your own.

> 📸 **Image placeholder:** `ui-ingame.png` — Running dialogue in PIE mode.
> *Setup:* PIE view with an active dialogue. At the bottom the dialogue widget: guard portrait on the left, name "Guard" in orange text, below it the dialogue text with an active typewriter effect (cursor visible). No choices visible (SayLine is currently running). Background: the level.

## Data flow of a single line

This is how a single SayLine travels through the system:

```text
Subsystem
  └─→ Instance::ContinueToNode(SayLine node)
        └─→ Node creates FMayDialogueMessage (text, speaker, tags)
              └─→ Instance broadcasts OnMessageReceived
                    ├─→ UI widget renders text + typewriter
                    ├─→ Audio system plays voice asset
                    └─→ Your quest system / analytics reacts (optional)
```

The events are **multicast delegates** — any number of systems can listen simultaneously without interfering with each other.

MayDialogue is ready to use immediately after installation without any further setup. GAS features are directly available if your project uses the Gameplay Ability System.

## Summary

| Building block | Lives | Responsible for |
| --- | --- | --- |
| Dialogue Asset | On disk / in Content Browser | Structure: nodes, speakers, variables |
| Participant Component | On the actor | Identity in the level, persistent memory |
| Subsystem | In the world (automatic) | Starting, managing, and ending conversations |
| UI Layer | During a dialogue | Displaying text, portraits, choices |

Next: [Graph & Visual Language](graph-visual-language.md).
