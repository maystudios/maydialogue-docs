---
description: Toolbar, tabs, compile button — everything you need to know about the asset editor window.
---

# The Asset Editor

Double-clicking a MayDialogue asset in the Content Browser opens the asset editor. Everything happens here: building the graph, triggering compilation, starting the preview, reading errors.

## Creating an asset

Three ways to create a new dialogue asset:

1. **Content Browser → right-click → Miscellaneous → MayDialogue Asset**
2. **Add button at the top left of the Content Browser → Miscellaneous → MayDialogue Asset**
3. **Duplicate an existing asset** (right-click → Duplicate, then rename)

Every new asset starts with an Entry node and empty Speakers/Variables lists.

> 📸 **Image placeholder:** `asset-editor-new-asset-context.png` — Right-click menu in the Content Browser, mouse hovering over "MayDialogue Asset".
> *Setup:* Content Browser in an empty folder. Right-click menu expanded, path Miscellaneous → MayDialogue Asset marked with a red arrow.

## The toolbar

Left to right:

| Button | Shortcut | What it does |
| --- | --- | --- |
| **Save** | `Ctrl+S` | Save the asset |
| **Compile** | `F7` | Run the validator + compiler |
| **Auto-Layout** | – | Automatically arrange the graph (Sugiyama) |
| **Play Dialog** | – | Start the Preview Runner |
| **Find** | `Ctrl+F` | Open and focus the Find Results tab |
| **Toggle Breakpoint** | `F9` | Set/remove breakpoint on the selected node |

Plus the standard Undo/Redo buttons (`Ctrl+Z` / `Ctrl+Y`).

> 📸 **Image placeholder:** `asset-editor-toolbar-annotated.png` — Toolbar with numbered red arrows on each button.
> *Setup:* Asset editor open. Toolbar area tightly cropped. Each button has a red number (1=Save, 2=Compile, 3=Auto-Layout, 4=Play, 5=Find, 6=Breakpoint).

## The tabs

The editor has **ten dock tabs**:

| Tab | Content | Default position |
| --- | --- | --- |
| **Graph** | Main work area with nodes and connections | Center |
| **Details** | Properties of the selected node | Right |
| **Compiler Results** | Errors and warnings after compile | Bottom |
| **Find Results** | Search results from Find-in-Dialogue | Bottom |
| **Palette** | All node types by category | Right |
| **Variables** | Variable declarations of the asset | Right |
| **Speakers** | Speaker list of the asset | Right |
| **Debugger Watch** | Variable watch during PIE | Bottom |
| **Preview** | In-editor dialogue preview | Right / expandable |
| **Outline** | All nodes as a searchable list | Left |

Tabs can be freely moved and docked. Your layout persists across sessions.

> 📸 **Image placeholder:** `asset-editor-tabs-overview.png` — Editor with an open dialogue, all tab headers visible. Compiler Results tab in the foreground with two warnings.
> *Setup:* Open asset `DA_Guard_Patrol` with an intentionally unconnected Exit pin so the compiler shows a warning. Show Compiler Results tab active.

## The Compile button

Press **Compile** (`F7`) after changing the graph. The process runs in two steps:

1. **Validator** checks for structural errors (missing nodes, unconnected pins, empty text, etc.).
2. If no errors: **Compiler** translates the graph into the runtime data structure.

If errors exist, the last correct compiled state remains active. You can continue testing the old state while fixing the errors.

{% hint style="info" %}
**Auto-Compile:** When `Auto Compile on Save` is active in the MayDialogue Editor settings (the default), the compiler runs automatically on save. You only need to press `F7` manually when you want immediate feedback.
{% endhint %}

### Compiler Results — what the error colors mean

| Color | Meaning |
| --- | --- |
| Red (Error) | Graph will **not** compile until the error is fixed |
| Yellow (Warning) | Graph compiles, but there are potential issues |

Clicking a line in the results list jumps directly to the affected node in the graph.

> 📸 **Image placeholder:** `asset-editor-compiler-results.png` — Compiler Results tab with one error (red) and one warning (yellow).
> *Setup:* Create a SayLine node without text (Error: "Empty Text") and a SayLine with no outgoing pin (Warning: "Unconnected Output"). Press Compile, show Results tab. Red arrow on the clickable node reference in the error row.

## Tab: Palette

The Palette lists all available node types by category:

- **Dialogue:** SayLine, PlayerChoice, RandomLine
- **Flow Control:** Branch, Wait, Link, SubGraph
- **Actions:** Camera Focus, Camera Shake, Play Animation, Apply Effect, Set Variable, Fire Event, Play Sound, Add Tag, Remove Tag, Trigger Cue
- **Special:** Exit, Knot

Drag and drop from the Palette into the graph. Alternatively: right-clicking on an empty area of the graph opens the same menu.

## Tab: Details

Shows the editable properties of the currently selected node. For SayLine nodes, the `SpeakerTag` field shows not a generic tag picker but a **dropdown listing all speakers defined in the asset** — including color chip and DisplayName.

> 📸 **Image placeholder:** `asset-editor-details-sayline.png` — Details panel of a selected SayLine.
> *Setup:* Select SayLine "Halt! Who are you?". Details panel shows: SpeakerTag dropdown with Guard entry (dark red color chip), DialogueText field filled, AdvanceModeOverride set to Manual, EmotionTags array empty. Red arrow on the speaker dropdown.

## Validator checks (quick reference)

| Check | Level | Meaning |
| --- | --- | --- |
| Empty Graph | Error | No nodes at all |
| Missing Entry | Error | No Entry node |
| Multiple Entries | Error | More than one Entry |
| Missing Exit | Warning | No Exit node (dialogue can run indefinitely) |
| Unconnected Pins | Error | Output pin with no connection |
| Empty Text | Error | SayLine or choice with no text |
| Unreachable Nodes | Warning | Nodes not reachable from the Entry |
| Missing Speaker | Error | SayLine with no valid speaker tag |
| Dead End | Error | Reachable node with no path to Exit |
| Variable Type Mismatch | Error | Same variable name with conflicting types |

Next: [Graph Panel →](graph-panel.md)
