---
description: How do I connect MayDialogue to my game code?
---

# Runtime Integration

Here you will learn how to control MayDialogue from your game code: starting dialogues, reacting to events, reading and writing variables.

## Three Ways to the Same Goal

MayDialogue offers three entry points. All of them ultimately start the same instance — you choose the approach that best fits your existing code.

| Approach | Class | When to use |
|---|---|---|
| **Participant component** | `UMayDialogueParticipant` | The NPC has the component on its actor; you want to start directly from the NPC Blueprint. Ideal for interaction triggers. |
| **Blueprint Library** | `UMayDialogueLibrary` | Quick one-liner from any Blueprint (Widget, GameMode, LevelScript). No reference caching needed. |
| **Subsystem directly** | `UMayDialogueSubsystem` | System code (Quest Director, Cutscene Director, Tutorial Manager). You need the subsystem for event binding anyway. |

> 📸 **Image placeholder:** `runtime-three-ways-overview.png` — Comparison diagram of the three methods side by side as three BP graph snippets.
> *Setup:* Three Blueprint graphs side by side in a screenshot. Left: NPC Blueprint with `Get Component by Class (MayDialogueParticipant)` → `Start Default Dialogue`. Center: any Blueprint with `Start Dialogue` Library Node (category MayDialogue). Right: `Get MayDialogue Subsystem` → `Start Dialogue` Subsystem Node.

## Chapter Overview

| Page | Content |
|---|---|
| [Starting a Dialogue](starting-dialogues.md) | All three methods with Blueprint graph and C++ snippet. |
| [Subsystem API](subsystem-api.md) | All subsystem functions with use case and example. |
| [Blueprint Library](library-api.md) | Library methods overview, Blueprint-focused. |
| [Bridge & Lifecycle Events](bridge-events.md) | Delegates: when they fire, how to bind. |
| [Read/Write API](read-write-api.md) | Read and write variables, select choices, ForceAdvance. |

## What Happens in the Background

You call `StartDialogue` — the rest runs automatically:

```text
Your code  →  Library::StartDialogue(Asset, Instigator, Target)
               ↓
           Subsystem::StartDialogue(...)
               ↓
           New Instance, execute Entry Node
               ↓  (Delegates)
           UI Widget ← OnMessageReceived / OnChoicesPresented
               ↓  (Player Input)
           Instance::AdvanceDialogue() / SelectChoice()
```

{% hint style="info" %}
The entire runtime API is accessible from Blueprint. You only need C++ when writing custom nodes or when you need tight system coupling without Blueprint overhead.
{% endhint %}
