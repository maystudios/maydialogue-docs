---
description: Auto-layout, cross-asset copy/paste, undo/redo, knots, comments and every shortcut at a glance.
---

# Comfort Features

The small helpers that make big dialogues manageable. Everything that makes the editor faster and easier to read lives here.

## Auto-Layout

**Toolbar → Auto-Layout** or `Ctrl+L` (if configured).

The graph is sorted by graph depth: Entry stays on the left, its direct successors one level further right, and so on. Within a level, nodes are arranged to minimize wire crossings. Connected nodes are additionally aligned vertically with one another (the Brandes–Köpf "Fast & Simple" method) so wires run as **straight** as possible instead of slanted; node height drives the spacing so nodes in a column never overlap. The result is balanced and fully deterministic (same graph → same layout).

When a node has **multiple outputs** (e.g. PlayerChoice, Branch, or RandomLine), its target nodes are arranged in **pin order** — the topmost output pin leads to the topmost target, the middle one to the middle, and so on (never in a random order).

When to use it:
- The graph is messy after many edits
- A new team member should be able to read the dialogue quickly
- Before a screenshot for the docs

When to touch up manually:
- Very large graphs with many parallel branches
- Comment boxes should be placed around thematic groups

> 📸 **Image placeholder:** `autolayout-before-after.png` — Before/after: chaotic graph on the left, the same graph after Auto-Layout on the right.
> *Setup:* Two parts. Left: an asset with 8 nodes scattered randomly, wires crossing multiple times. Right: the same asset after clicking Auto-Layout — Entry on the left, nodes arranged neatly in three vertical columns, no wire crossings. Red arrow pointing at the Auto-Layout button in the toolbar (between the two variants).

## Auto-Reroute

**Toolbar → Auto-Reroute** (toggle; also under *Project Settings ▸ Plugins ▸ MayDialogue Editor ▸ Layout*).

Long edges that skip several layers would otherwise cut across the graph as one long diagonal. Auto-Reroute makes Auto-Layout insert **aligned reroute (knot) nodes** along such edges, so the wire runs in clean segments through the layout's routing channels — the way you would route it by hand. **On by default.**

- **Minimal:** a knot is added **only where a wire actually bends**. Edges the layout already routes straight stay a single clean wire — the graph is never littered with unnecessary reroutes.
- **On/off** via the toolbar toggle (or *Project Settings ▸ … ▸ Layout*); then click **Auto Layout** to apply it.
- Auto-generated reroutes are **rebuilt on every Auto-Layout run** (idempotent — they never pile up); **your own** reroute nodes are left untouched.
- Runtime-transparent: the compiler collapses knot chains anyway, so the dialogue behaviour does not change.

## Auto-Grid

**Toolbar → Auto-Grid** (toggle) plus the **Grid Size** dropdown right next to it.

By default the graph editor uses Unreal's fine 16 px grid. Auto-Grid switches to a **coarser grid** (16/32/64/128/256 px) that nodes align to automatically — when **dragging, pasting, creating, and during auto-layout**. The background grid is drawn at the chosen size, so it's immediately clear what nodes snap to.

- **On/off** via the toolbar toggle; the setting is project-wide and persists.
- **Size** via the dropdown or under *Project Settings ▸ Plugins ▸ MayDialogue Editor ▸ Auto-Grid*.
- **Off = unchanged default behaviour** (the engine's 16 px grid).

When to use it: when you want to keep nodes tidily lined up by hand without having to aim pixel-perfectly — the larger grid snaps nodes flush automatically.

## Comment Boxes

Select multiple nodes and press `C`. A coloured box frames the selection.

- Adjust the **title** and **colour** in the Details panel after selecting the box.
- **Moving** the box moves all contained nodes with it.
- Change the **size** by dragging the edge.

Sensible groups for comments:
- *"Greeting"* — first conversation section
- *"Confrontation"* — escalation branches
- *"Endings"* — all exit paths

> 📸 **Image placeholder:** `comfort-comment-boxes.png` — Graph with two coloured comment boxes and visible titles.
> *Setup:* Graph with a green comment box "Greeting" (Entry + two SayLines) and a red comment box "Confrontation" (PlayerChoice + two SayLines + Branch). Both boxes with visible titles at the top. Connecting wires run between the boxes.

## Knots (Reroute Points)

Double-click an **existing wire** to insert a knot point.

- Knots have multiple inputs and outputs.
- In the editor: they help route long wires around other nodes.
- At runtime: they don't exist — the compiler resolves all knot chains.

Use knots when:
- A wire obscures other nodes
- Two far-apart nodes need to be connected
- You want to visually split a wire (one output → multiple targets)

> 📸 **Image placeholder:** `comfort-knots.png` — Two knot points on a long wire routed around a comment block.
> *Setup:* A SayLine node top-left connected to an Exit node bottom-right, the wire arcing around a comment box in the middle. Two knots mark the bend points. Red arrow pointing at a knot point labelled "Double-click on wire".

## Undo / Redo

`Ctrl+Z` / `Ctrl+Y`

Applies to all graph actions:
- Adding and deleting nodes
- Drawing and breaking connections
- Moving nodes
- Changing properties in the Details panel
- Adding and removing sub-nodes
- Creating and moving comment boxes

## Copy / Cut / Paste

| Shortcut | Effect |
| --- | --- |
| `Ctrl+C` | Copy selected nodes |
| `Ctrl+X` | Cut |
| `Ctrl+V` | Paste (with offset) |
| `Ctrl+D` | Duplicate (in one step, with an automatic offset) |

### Cross-Asset Copy/Paste

You can copy nodes from one dialogue asset and paste them into another:

1. Select the nodes in the source asset.
2. `Ctrl+C`.
3. Open the target asset.
4. `Ctrl+V`.

On paste, all nodes get new GUIDs. Connections **within** the copied selection are preserved. Connections to nodes outside the selection are dropped.

> 📸 **Image placeholder:** `comfort-cross-asset-paste.png` — Two asset editor windows side by side. Left: three selected nodes. Right: the same after the paste, nodes inserted.
> *Setup:* Two editor windows side-by-side. Left `DA_Guard_Day`, three nodes selected (blue selection frame). Right `DA_Guard_Night`, the three nodes already pasted (slightly offset, connections between them preserved). Red arrow showing the copy direction from left to right.

## Speaker Dropdown in the Details Panel

In the properties of every node with a `SpeakerTag` field, you get **not a generic tag picker** but a dropdown with:
- The speaker's colour chip
- DisplayName
- The tag as a tooltip

This saves typing and typos. Speakers are picked from a short list instead of typing tags blind.

## SubGraph Breadcrumbs

When you double-click a SubGraph node, the sub-graph opens. The breadcrumb path appears at the top:

```
DA_MainDialogue > IntroGraph > CombatResponse
```

Clicking a segment jumps back there.

## Drag-and-Drop from the Palette

Instead of right-click → context menu: open the Palette tab and drag a node type into the graph. Handy when you know which type you want and filtering the context menu gets slow.

## Full Shortcut Reference

### General

| Shortcut | Effect |
| --- | --- |
| `Ctrl+S` | Save |
| `F7` | Compile |
| `Ctrl+F` | Open Find-in-Dialogue |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` | Redo |

### Graph Editing

| Shortcut | Effect |
| --- | --- |
| `Ctrl+C` | Copy |
| `Ctrl+X` | Cut |
| `Ctrl+V` | Paste |
| `Ctrl+D` | Duplicate |
| `Delete` | Delete selected nodes |
| `C` | Create a comment box around the selection |

### Navigation

| Shortcut | Effect |
| --- | --- |
| `F` | Zoom to selected nodes |
| `Home` | Center on the Entry node |
| Right-click + drag | Pan the graph |
| Mouse wheel | Zoom |

### Debugger

| Shortcut | Effect |
| --- | --- |
| `F9` | Toggle breakpoint |
| `F5` | Continue |
| `F10` | Step Over |
| `F11` | Step Into |
| `Shift+F11` | Step Out |

> 📸 **Image placeholder:** `comfort-shortcuts-overview.png` — Shortcut cheat sheet as an overlay or popup in the editor (if it exists), otherwise the toolbar area with tooltips visible.
> *Setup:* If the editor has a shortcut-help popup: a screenshot of it. Alternatively: the toolbar with an open tooltip on the Compile button showing the F7 shortcut.

> 📸 **Image placeholder:** `comfort-duplicate-workflow.png` — Three SayLine nodes, one selected, a duplicate appearing next to it after Ctrl+D.
> *Setup:* The middle SayLine node selected (blue frame). After Ctrl+D: the duplicate appears directly to its right, also selected, slightly offset. Red arrow pointing at the duplicate.

---

End of the editor chapter. Continue with the **Node Reference**: [Core Nodes →](../nodes/README.md)
