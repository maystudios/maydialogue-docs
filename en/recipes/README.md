---
description: Ready-made solution patterns for recurring dialogue tasks — reproducible in 5–20 minutes.
---

# Recipes

The recipe gallery collects **ready-made solution patterns** for recurring dialogue tasks. Each recipe is structured so you can reproduce it in about 10–20 minutes and transfer it to your project.

## All Recipes at a Glance

| # | Recipe | Difficulty | Tags |
|---|--------|------------|------|
| 1 | [Simple NPC Conversation](simple-npc-talk.md) | Beginner | Basics |
| 2 | [Random Greetings](random-greetings.md) | Beginner | Branching |
| 3 | [Branching with Conditions](branching-conditions.md) | Beginner | Branching, GAS |
| 4 | [Reusable Dialogue Fragments](linking-dialogues.md) | Beginner | Structure |
| 5 | [SubGraph Organization](subgraph-organization.md) | Beginner | Structure |
| 6 | [Choice Visible Only with Tag](choice-with-tag-requirement.md) | Beginner | Branching, GAS |
| 7 | [Choice with Attribute Condition](choice-with-attribute-requirement.md) | Beginner | Branching, GAS |
| 8 | [GAS-Driven Dialogue](gas-driven-dialogue.md) | Advanced | GAS |
| 9 | [Apply Gameplay Effect from Dialogue](apply-gameplay-effect.md) | Advanced | GAS |
| 10 | [Read Quest Status in Dialogue](quest-status-in-dialogue.md) | Advanced | GAS, Branching |
| 11 | [Dialogue Sets Quest Progress](dialogue-sets-quest-progress.md) | Advanced | GAS |
| 12 | [Track Relationship Score](relationship-counter.md) | Advanced | Persistence |
| 13 | [NPC Remembers Second Meeting](npc-remembers-meeting.md) | Advanced | Persistence |
| 14 | [Multilingual Dialogue](multilingual-dialogue.md) | Advanced | Audio |
| 15 | [Inner Monologue with 2D Audio](inner-monologue-2d.md) | Advanced | Audio |
| 16 | [Camera Pan on Speaker](camera-pan-on-speaker.md) | Advanced | Camera |
| 17 | [Jump Scare with Camera Shake](jump-scare-shake.md) | Advanced | Camera, Animation |
| 18 | [NPC Animation During Line](npc-animation-during-line.md) | Beginner | Animation |
| 19 | [Timed Choice (Auto-Select)](timed-choice.md) | Beginner | Branching, UI |
| 20 | [Wait for External GameplayEvent](wait-for-event.md) | Advanced | GAS |
| 21 | [Attach Custom UMG Widget](custom-umg-widget.md) | Advanced | UI, Extension |
| 22 | [Build Custom Requirement in Blueprint](custom-blueprint-requirement.md) | Advanced | Extension |

## Recommended Order for Beginners

If you are using MayDialogue for the first time, this path is recommended:

```text
1. Simple NPC Conversation
       │
       ├─► Random Greetings
       │
       └─► Branching with Conditions
                   │
                   ├─► Choice Visible Only with Tag
                   ├─► Choice with Attribute Condition
                   └─► GAS-Driven Dialogue
                               │
                               ├─► Read Quest Status in Dialogue
                               ├─► Dialogue Sets Quest Progress
                               └─► Apply Gameplay Effect

In parallel (structure):
       Reusable Fragments → SubGraph Organization

Advanced (as needed):
       Relationship Score → NPC Remembers
       Camera → Jump Scare → NPC Animation
       Timed Choice → Wait for Event
       Custom Widget → Custom Requirement
```

## Prerequisites

The recipes are written so you can jump right in. When a concept is unclear, the relevant concept pages are linked in each recipe.

{% hint style="info" %}
All recipes assume you have integrated the plugin per the [Installation](../getting-started/installation.md) instructions and have a dialogue widget configured in [Project Settings](../getting-started/project-settings.md).
{% endhint %}

## Conventions in Code Snippets

| Abbreviation | Meaning |
|--------|---------|
| `Sub` | `UMayDialogueSubsystem*` |
| `Inst` | `UMayDialogueInstance*` |
| `Asset` | `UMayDialogueAsset*` |
| `PC` | `APlayerController*` |
| `NPC` | Conversation partner actor |

Full boilerplate can be found in [Runtime → Starting a Dialogue](../runtime/starting-dialogues.md).

## Further Topics Outside the Recipes

- **Writing custom nodes** → [Extension → Custom Nodes](../extension/custom-nodes.md)
- **SaveGame integration** → [Persistence](../persistence/README.md)
- **UI theming** → [UI → Themes & Starter Kits](../ui/themes.md)
- **Audio fallback chain** → [Audio → Three-Level Fallback](../audio/three-level-fallback.md)
