---
description: All important MayDialogue terms explained in alphabetical order.
---

# Glossary

This directory defines the technical terms used in the documentation and in the editor. If you see a term in the graph or in an error and don't know what it means, look it up here.

---

### Advance Mode

Determines how the dialogue transitions from the current node to the next. Available modes:

| Mode | Behavior |
| --- | --- |
| `Manual` | Player input (e.g. pressing a key) |
| `Timer` | Automatically after a configured time |
| `AfterVoice` | Waits until the voice asset has finished playing |
| `AfterAnimation` | Waits for the end of a montage |
| `Immediate` | Transitions immediately without pausing |

Each SayLine node can override the global Advance Mode per node (`AdvanceModeOverride`).

---

### Asset

A `UMayDialogueAsset` — the Blueprint asset you create in the Content Browser and open in the dialogue editor. It contains the complete graph (nodes, links), all speaker definitions, and the variable declarations for the dialogue.

---

### Bridge

`IMayDialogueBridge` — an interface that allows external systems (e.g. quest system, inventory, custom logic) to connect to a running dialogue. Through the Bridge you can programmatically select choices, fire events, and read variables. Useful when integrating a dialogue system into a larger game loop.

---

### Choice

A single answer option in a PlayerChoice node. Each Choice can have Requirements, tags, and its own output pin. Choices are displayed as sub-nodes inside the PlayerChoice node.

See also: [FailedAndHidden](#failedandhidden), [FailedButVisible](#failedbut-visible), [Player Choice](#player-choice).

---

### Cue (MayDialogue Lifecycle Cue)

Fired via the plugin settings as a GameplayCue tag per lifecycle event (Started/Ended/NodeReached/...). The `TriggerCue` Action Node also fires a freely chosen GameplayCue. In both cases, this goes through the real GAS GameplayCue system.

---

### Dialogue Scope

Variables that only exist during the running dialogue Instance. When the dialogue ends (or is aborted), Dialogue-scope variables are discarded. For persistent values, use [Participant Scope](#participant-scope).

---

### Emotion Tag

An `FGameplayTagContainer` field on SayLine nodes. Emotion tags are hierarchical meta-tags that describe how a line is performed — e.g. `Dialogue.Emotion.Angry`, `Dialogue.Emotion.Scared`. The widget, audio system, and animations can evaluate emotion tags to adjust the appearance of the line.

---

### Entry

The start node of a dialogue asset. Every asset has exactly one Entry node. It has no input pin and one output pin that points to the first content node. If the Entry node is missing, the compile fails.

---

### Exit

The end node of a dialogue asset or sub-graph. It has one input pin and no output pin. When the dialogue flow reaches the Exit, the Instance ends (or, for sub-graphs: returns to the calling graph). Exit nodes can carry a completion status (e.g. `Completed`, `Failed`).

---

### FailedAndHidden

A Requirement result: the condition is not met, and the Choice or node is completely invisible to the player. If no other path is offered, a PlayerChoice node can appear empty.

---

### FailedButVisible

A Requirement result: the condition is not met, but the Choice is visible in the widget — locked, with visual feedback (e.g. grey button, lock icon). The player can see the option but cannot select it.

---

### Instance

A `UMayDialogueInstance` — the running object of a dialogue. Created when the dialogue starts, it manages the current node pointer, variables, and the scope stack. Destroyed when the dialogue ends (Exit node) or is aborted. There is exactly one Instance per running dialogue.

---

### Link

A node type that jumps the dialogue flow into another dialogue asset. Links allow you to structure dialogues modularly and navigate between assets. Optionally, the flow returns to the calling asset after the target asset ends (`bReturnAfterExit`).

---

### Node

A single element in the dialogue graph. Nodes are the building blocks of a dialogue — from SayLines to Choices to Action nodes. Each node has input and output pins through which the flow is connected. Nodes can contain sub-nodes (Requirements, SideEffects, Choices).

---

### Participant

A `UMayDialogueParticipant` component attached to an actor. It identifies the actor as a dialogue participant via a `ParticipantTag` and stores Participant-scope variables. Without a Participant component, an actor cannot take a role in the dialogue.

---

### Participant Scope

Variables attached to a Participant component that survive across individual dialogues. Optionally, Participant-scope variables can be saved in a SaveGame. Contrast: [Dialogue Scope](#dialogue-scope).

---

### Player Choice

A node type that presents the player with one or more answer options. The dialogue pauses until the player selects a Choice. Each Choice has its own output pin that routes the flow in the corresponding direction.

---

### Preview Runner

`FMayDialoguePreviewRunner` — an embedded playback system in the asset editor. Plays a dialogue directly in the editor without needing to start PIE. Enables rapid testing of text, structure, and branch logic. Tags can be simulated to test Requirements.

See: [Debugging Tips → Preview Runner](../troubleshooting/debugging-tips.md#preview-runner-iteration-ohne-pie).

---

### Requirement

A condition that must be met for a node or Choice to execute or be displayed. Requirements are sub-nodes inside node bodies. The result of a Requirement check is `Passed`, `FailedButVisible`, or `FailedAndHidden`.

Custom Requirement types can be created as Blueprint subclasses of `UMayDialogueRequirement`.

---

### SayLine

The most common node type. A SayLine displays a speaker's text, optionally plays audio, and waits for an advance (depending on the Advance Mode). The title bar of a SayLine node has the color of the assigned speaker.

---

### SideEffect

A sub-node that performs a side action when a node is entered or exited — e.g. setting a variable, firing an event, adding a tag. SideEffects have no result that affects the flow.

---

### Speaker

`FMayDialogueSpeaker` — an asset-level definition of a speaker. Contains: display name, portrait texture, node color, voice asset mapping, and Babel profile. Speakers are defined in the Speakers panel of the dialogue asset and referenced on nodes via a speaker tag.

---

### SubGraph

A partial graph within the same dialogue asset. SubGraphs allow you to structure reusable dialogue sections without creating a separate asset. Entered via a SubGraph node, returned via its Exit.

---

### Subsystem

`UMayDialogueSubsystem` — the central world instance that manages all running dialogues. It starts and stops dialogues, resolves participants, and provides delegates for dialogue events. Access via `GetWorld()->GetSubsystem<UMayDialogueSubsystem>()` or the Blueprint function node `GetMayDialogueSubsystem`.

---

### Typewriter

An optional text effect that reveals SayLine text character by character. Configurable via `TypewriterCharsPerSecond` at the speaker level. Babel voices are synchronized with the typewriter — Babel blips on each revealed character.

---

### Variables

A set of declared key-value pairs in a dialogue asset. Variables have a type (Bool, Int, Float, String, Tag), a scope (Dialogue or Participant), and a default value. They are created in the Variables panel of the asset and written via SetVariable nodes and read in Requirements.

---

> 📸 **Image placeholder:** `glossary-speakers-panel.png` — Speakers panel of a dialogue asset with two defined speakers.
> *Setup:* Dialogue asset open, Speakers tab active. Table shows two entries: speaker "Guard" (Tag: `Dialogue.Speaker.Guard`, color: dark red, portrait: portrait texture assigned) and speaker "Player" (Tag: `Dialogue.Speaker.Player`, color: blue, portrait: empty).
