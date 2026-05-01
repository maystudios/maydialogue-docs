---
description: From an empty project to a playable dialogue in three steps.
---

# Getting Started

MayDialogue is a dialogue plugin for Unreal Engine 5.7. You get a visual graph editor, a ready-made UI layer, 3D audio with a procedural Babel fallback, camera controls, typewriter effects, and GAS integration. Everything ships in one plugin with no external dependencies.

> 📸 **Image placeholder:** `getting-started-overview.png` — Screenshot of the MayDialogue editor with an open dialogue asset.
> *Setup:* Asset `DA_Gate_Guardian` open in the editor. Visible: the graph with several colored nodes (Entry green, SayLines dark red/grey, PlayerChoice wide, Exit red), the Speakers panel on the right with two entries, the Outline panel on the left with a node list. Caption: "The graph is the document."

## What you need

* **Unreal Engine 5.7** (binary or source build)
* An existing project (Blueprint or C++ project)
* Basic knowledge of Blueprint or C++. You don't need to know the plugin internals.

## The three steps

### 1: Installation

Copy the plugin folder into your project, regenerate project files, open the editor. Takes about five minutes.

[→ Installation](installation.md)

### 2: Quick Start

A playable NPC dialogue in your level in five minutes. No UMG setup, no audio configuration, no input handling.

[→ Quick Start](quick-start.md)

### 3: Walkthrough

A complete dialogue with variables, GAS attribute checks, choice requirements, and SideEffect actions.

[→ Walkthrough](first-dialogue.md)

## Recommended order

```
Installation → Quick Start → Walkthrough → Project Settings → Core Concepts
```

From there you can dive into whichever topics your project needs: [UI](../ui/README.md), [Audio](../audio/README.md), [GAS](../gas/README.md), or [Recipes](../recipes/README.md).

> 📸 **Image placeholder:** `getting-started-flow-diagram.png` — Overview of the recommended learning paths.
> *Setup:* Simple graphic with five boxes left to right: "Installation" → "Quick Start" → "Walkthrough" → "Project Settings" → "Core Concepts". Each box labeled, arrows between them.

{% hint style="warning" %}
**Blueprint-only project?** You can use the plugin as a binary distribution without compiling it yourself. As soon as you write custom nodes, requirements, or side effects in C++, your project needs a C++ module and a rebuild.
{% endhint %}
