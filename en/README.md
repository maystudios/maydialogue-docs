---
description: >-
  A dialogue plugin for Unreal Engine 5.7 with visual graph editor,
  3D audio, typewriter, camera controls, GAS integration, and a ready-made
  UI set — ready to use out of the box, no boilerplate required.
---

# Welcome to MayDialogue

**MayDialogue** is a complete dialogue system for Unreal Engine 5.7. You don't need to set up audio routing, build a UMG hierarchy, or write input mode logic. Enable the plugin, add a component to your NPC, assign a dialogue asset, call the start function — audio plays in 3D, the widget appears, text runs with a typewriter effect, choices are clickable, and the dialogue ends cleanly.

{% hint style="success" %}
**Five minutes from zero to playable.** No Blueprint boilerplate, no UI setup. Check out the [Quick Start](getting-started/quick-start.md).
{% endhint %}

> 📸 **Image placeholder:** `hero-running-dialogue.png` — Screenshot of a running dialogue in-game.
> *Setup:* PIE session in a test level. Visible: an NPC in the foreground, dialogue widget at the bottom of the screen with speaker name "Guard", portrait on the left, text "Halt! Who are you?" with an active typewriter effect (mid-animation), two choice buttons "A friend of the king." / "That's none of your business." Skip hint "Press Space" on the right edge.

## What you get

<table data-view="cards">
  <thead>
    <tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Visual Graph Editor</strong></td>
      <td>Speaker colors, inline text, Sub-Nodes as pills in the body — the graph reads like a screenplay.</td>
      <td><a href="concepts/graph-visual-language.md">graph-visual-language.md</a></td>
    </tr>
    <tr>
      <td><strong>Immediately Playable UI</strong></td>
      <td>Slate debug widget out of the box; swappable UMG component set for production.</td>
      <td><a href="ui/README.md">UI System</a></td>
    </tr>
    <tr>
      <td><strong>3D Audio with Fallback</strong></td>
      <td>Voice plays automatically at the speaker's actor. Three override levels (plugin / speaker / node).</td>
      <td><a href="audio/README.md">Audio System</a></td>
    </tr>
    <tr>
      <td><strong>GAS Integration</strong></td>
      <td>Tag, attribute, and ability requirements; ApplyEffect, AddTag, RemoveTag, TriggerCue as nodes.</td>
      <td><a href="gas/README.md">GAS Integration</a></td>
    </tr>
    <tr>
      <td><strong>Preview Without PIE</strong></td>
      <td>Play a dialogue directly from the asset editor, with simulated GAS state and culture switching.</td>
      <td><a href="editor/preview-runner.md">Preview Runner</a></td>
    </tr>
    <tr>
      <td><strong>Robust and Extensible</strong></td>
      <td>Production architecture following Epic best practices. Custom nodes, requirements, and side effects via Blueprint.</td>
      <td><a href="extension/README.md">Extending</a></td>
    </tr>
  </tbody>
</table>

## Who is this for?

MayDialogue is a great fit for:

* **Single-player story games** — horror, visual novel, RPG, walking simulator, narrative adventure.
* **Indie teams and game jams** — productive in minutes, no weeks of setup.
* **Projects using GAS** — tags, attributes, and effects speak the same language as your characters.
* **Beginners with no C++ experience** — all major workflows work in Blueprint.
* **Experienced C++ developers** — subsystem API, delegates, and C++ subclasses for custom logic.

MayDialogue is **not**:

* A quest or mission system (read quest status via tags / variables; the quest tracker itself lives elsewhere).
* A voice recording or TTS tool (the plugin plays finished `USoundBase` assets).
* A Sequencer replacement (for longer cutscenes, start Level Sequences that can in turn trigger MayDialogue dialogues).

## Where should I start?

{% content-ref url="getting-started/quick-start.md" %}
[quick-start.md](getting-started/quick-start.md)
{% endcontent-ref %}

{% content-ref url="getting-started/first-dialogue.md" %}
[first-dialogue.md](getting-started/first-dialogue.md)
{% endcontent-ref %}

{% content-ref url="recipes/README.md" %}
[recipes/](recipes/README.md)
{% endcontent-ref %}

## Reading Paths

Choose the path that suits you:

| You are… | Recommended order |
| --- | --- |
| **Beginner without C++** | [Installation](getting-started/installation.md) → [Quick Start](getting-started/quick-start.md) → [Walkthrough](getting-started/first-dialogue.md) → [Recipes](recipes/README.md) |
| **Designer** | [Quick Start](getting-started/quick-start.md) → [Graph & Visual Language](concepts/graph-visual-language.md) → [Editor Tour](editor/README.md) → [Recipes](recipes/README.md) |
| **Blueprint Developer** | [Walkthrough](getting-started/first-dialogue.md) → [Runtime Integration](runtime/README.md) → [Bridge & Events](runtime/bridge-events.md) → [Custom Nodes](extension/custom-nodes.md) |
| **C++ Developer** | [Architecture](concepts/architecture.md) → [Subsystem API](runtime/subsystem-api.md) → [Reference](reference/README.md) → [Extension](extension/README.md) |
| **UI Designer** | [UI Overview](ui/README.md) → [UMG Architecture](ui/umg-architecture.md) → [Themes](ui/themes.md) → [Rich Text Tags](ui/rich-text-tags.md) |

## Status

| Information | Value |
| --- | --- |
| Engine | Unreal Engine 5.7 |
| Plugin Version | 0.1.0 (Beta) |
| Last Updated | 2026-04-27 |
| Dependencies | `GameplayAbilities`, `GameplayTags`, `StructUtils`, `EnhancedInput` |

Current limitations are listed under [Known Issues](troubleshooting/known-issues.md). New here? Start with the [Quick Start](getting-started/quick-start.md).

---

> 📸 **Image placeholder:** `hero-graph-overview.png` — Overview photo of a complete dialogue graph.
> *Setup:* Example asset `DA_Demo_VillagerEncounter` open in the asset editor. Auto-layout applied. Visible: Entry node green on the left, three speaker-colored SayLines (two red guard lines, one grey player line), a PlayerChoice node with three Choice Sub-Nodes, a Branch node with a Requirement Sub-Node "HasTag(Story.Found.Codex)", two reaction SayLines, an Exit node red on the right. Comment box "Greeting" around the first three nodes. Outline tab on the left visible with the node list.
