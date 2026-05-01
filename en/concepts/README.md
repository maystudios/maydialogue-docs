---
description: The mental models you need before building nodes.
---

# Core Concepts

This section lays the foundation. Understanding these concepts explains why the plugin works the way it does — and saves you frustration when building complex dialogues.

## The four foundational ideas

1. **The graph is the document.** A dialogue asset reads like a screenplay: who speaks, what do they say, what choice does the player have, and what follows from that. Not a property list.
2. **The running conversation is its own instance.** The asset is the blueprint; every started conversation lives as its own instance with a clearly defined lifecycle — start, traversal, end, cleanup.
3. **Participants are the actors in the level.** Every actor that speaks or listens in a dialogue carries a Participant component. It is their identity in the conversation.
4. **Sub-Nodes keep logic compact.** Requirements, Choices, and SideEffects are not separate graph boxes; they are pills in the body of a parent node.

## Chapters in this section

| Page | Question it answers |
| --- | --- |
| [Architecture Overview](architecture.md) | What are the main building blocks of the plugin and how do they connect? |
| [Graph & Visual Language](graph-visual-language.md) | How do I read the graph? Colors, shapes, Sub-Nodes. |
| [Instance & Lifecycle](instance-lifecycle.md) | What happens from start to exit — and what happens on abort? |
| [Participants & Speakers](participants-speakers.md) | Participant component vs. speaker definition in the asset. |
| [Variables & Scopes](variables-scopes.md) | Dialogue scope or participant scope — when do I use which? |
| [Emotions & Tags](emotions-tags.md) | Setting tags on SayLines and letting the UI and audio react. |

## Quick entry for readers in a hurry

If you want to get a dialogue running as quickly as possible, read [Architecture Overview](architecture.md) and [Instance & Lifecycle](instance-lifecycle.md) first. These two pages give you the essential big picture.

For everyone else: the chapter order is designed so that each one builds on the previous. Reading top to bottom works best.
