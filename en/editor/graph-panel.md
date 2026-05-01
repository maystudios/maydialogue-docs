---
description: Navigation, placing nodes, drawing connections, inline editing, Sub-Nodes, knots, and comments.
---

# The Graph Panel

The graph is the main work area. Here you build the flow of your dialogue — left to right (Entry to Exit), node by node.

> 📸 **Image placeholder:** `graph-panel-overview.png` — Overview of a medium-sized dialogue in the graph.
> *Setup:* Open asset `DA_Guard_Greeting`. Visible: Entry node (green), three SayLine nodes (dark red title bars, speaker "Guard"), one PlayerChoice node (blue) with two Choice Sub-Nodes, two more SayLines as reactions, one Exit node (red). All nodes connected. No zoom — full graph fits in the image.

## Adding nodes

### Right-click menu (fastest way)

Right-click on an **empty area of the graph** to open the context menu. Start typing immediately — the list filters live as you type.

Node categories in the menu:

| Category | Node types |
| --- | --- |
| Dialogue | SayLine, PlayerChoice, RandomLine |
| Flow Control | Branch, Wait, Link, SubGraph |
| Actions | Camera Focus, Camera Shake, Play Animation, Apply Effect, Set Variable, Fire Event, Play Sound, Add Tag, Remove Tag, Trigger Cue |
| Special | Exit, Knot |

> 📸 **Image placeholder:** `graph-contextmenu.png` — Right-click menu on empty graph area, "SayLine" typed in the search field.
> *Setup:* Right-click on empty area next to the Entry node. Search field shows "SayLine", one match highlighted below. Red arrow on the search field.

### Drag and drop from the Palette

Alternatively: open the Palette tab, drag a node type into the graph.

## Connecting nodes

1. **Click an output pin** and hold the mouse button.
2. Drag the wire to the **input pin** of the target node.
3. Release the mouse button — the connection is made.

**Drag onto empty area**: releasing the wire on an empty spot opens the context menu with compatible target nodes.

{% hint style="info" %}
The graph only allows connections between compatible dialogue pins. Input→Input or Output→Output does not work.
{% endhint %}

## Navigation

### Panning the camera

| Method | Action |
| --- | --- |
| Right-click + drag | Pan the graph |
| Middle-click + drag | Pan the graph |
| WASD | Pan the graph (after clicking on empty area) |

### Zoom

- **Mouse wheel** to zoom in/out.
- Current zoom level displayed in the lower right.

### Quick navigation

| Shortcut | Effect |
| --- | --- |
| `F` | Zoom to selected nodes |
| `Home` | Center camera on Entry node |

### SubGraph navigation

Double-clicking a SubGraph node opens the sub-graph. A **breadcrumb path** appears at the top:

```
ParentAsset > MainGraph > SubGraphName
```

Clicking a segment jumps back to it.

> 📸 **Image placeholder:** `graph-breadcrumb.png` — Breadcrumb bar at the top of the graph with two levels: "DA_Guard_Greeting > CombatResponse".
> *Setup:* Double-click a SubGraph node. Breadcrumb shows parent asset and sub-graph name. Red arrow on breadcrumb.

## Selecting and editing nodes

| Action | How |
| --- | --- |
| Select node | Single click |
| Select multiple | Shift+click or rubber-band selection (drag on empty area) |
| Move node | Select and drag |
| Delete node | `Delete` |
| Duplicate node | `Ctrl+D` |

### Inline text editing

For **SayLine nodes**: double-clicking directly on the node's text body opens an inline text field. Enter text and confirm with `Enter` or by losing focus.

You don't need to switch to the Details panel to change text.

> 📸 **Image placeholder:** `graph-inline-edit.png` — SayLine node with an active inline text field, cursor blinking in the text.
> *Setup:* Double-click SayLine node. The text field in the node body is active, blinking cursor visible. Red arrow pointing at the inline text field.

## Adding Sub-Nodes

Some nodes have **Sub-Nodes** (Requirements, Choices, SideEffects). Two ways:

**Way 1 – right-click on the node:**
Right-click on a PlayerChoice or Branch node → *"Add Requirement"* / *"Add SideEffect"* / *"Add Choice"*.

**Way 2 – Details panel:**
1. Select the node.
2. Details panel → array `Requirements`, `Choices`, or `SideEffects`.
3. Click **Add Element** → choose a type from the dropdown.

> 📸 **Image placeholder:** `graph-subnodes.png` — PlayerChoice node with two Choice Sub-Nodes and one Requirement Sub-Node (visible as pills on the node edge).
> *Setup:* PlayerChoice with two choices "A friend of the king." and "That's none of your business." plus one HasTag requirement. All Sub-Nodes visible as compact pills below the node title.

## Knots (reroute points)

Double-clicking an **existing wire** inserts a knot point. This lets you route long connections cleanly around other nodes.

- Knots only exist in the editor — the compiler resolves them completely.
- Knots support multiple inputs and outputs.

## Comment boxes

Select multiple nodes and press `C` → A colored comment box frames the selection.

- Change **title** and **color** in the Details panel.
- Moving the box moves all contained nodes with it.
- Use comments for semantic groups: *"Greeting"*, *"Confrontation"*, *"Endings"*.

> 📸 **Image placeholder:** `graph-comments-knots.png` — Graph with two comment boxes (green: "Greeting", red: "Confrontation") and a knot on a long wire.
> *Setup:* Three SayLines in a green comment box "Greeting", two more in a red comment box "Confrontation". Connection wire between the boxes routed around with a knot at the top.

## Copy / Paste (also cross-asset)

| Shortcut | Effect |
| --- | --- |
| `Ctrl+C` | Copy selected nodes |
| `Ctrl+X` | Cut |
| `Ctrl+V` | Paste |

**Cross-asset paste works:** paste nodes from one dialogue asset into another. Connections within the copied selection are preserved, all nodes receive new GUIDs.

## Setting breakpoints in the graph

Right-click on a node → **Toggle Breakpoint** (or `F9` with the node selected). A small red dot appears in the upper right of the node.

Breakpoints are used for the [PIE Debugger](debugger.md) — execution pauses when this node is reached.

> 📸 **Image placeholder:** `graph-breakpoint.png` — Branch node with a red breakpoint dot in the upper right.
> *Setup:* Branch node selected, F9 pressed. Red dot clearly visible. Red annotation arrow pointing at the dot with label "Breakpoint".

Next: [Speakers Panel →](speakers-panel.md)
