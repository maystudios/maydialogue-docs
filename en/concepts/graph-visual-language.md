---
description: How to read the dialogue graph — colors, shapes, Sub-Nodes, choice pins.
---

# Graph & Visual Language

When you open a dialogue asset, the first thing you see is the graph. This section explains what you are looking at — so you can read an unfamiliar dialogue immediately, without clicking every node individually.

> 📸 **Image placeholder:** `graph-overview-full.png` — Overview of a medium-sized dialogue graph.
> *Setup:* Asset `DA_Guard_Greeting` open. Visible left to right: `Entry` (green capsule) → `SayLine "Halt! Who are you?"` (dark red title bar, speaker: Guard) → `PlayerChoice` (purple, two choice pills in the body) → two outgoing wires each to a `SayLine` reaction → both end at `Exit` (red capsule). No comment overlay, clean horizontal layout.

## Principles at a glance

- **Color = speaker.** You identify who is speaking from the title bar color, not from the text.
- **Text is visible inline.** You don't need to click a node to know what it says.
- **Sub-Nodes are pills in the body.** Requirements, Choices, and SideEffects appear directly on the parent node — no separate boxes in the graph.
- **Shapes indicate semantics.** Entry/Exit are capsules, Branch is diamond-shaped, SubGraph has a folder icon.

## Anatomy: SayLine node

```
┌────────────────────────────────────────────┐
│ [dark red] Guard                           │  ← Title bar in speaker color
├────────────────────────────────────────────┤
│ Halt! Who are you?                         │  ← Inline text (editable via double-click)
│                                            │
│ 🏷 Dialogue.Emotion.Stern                   │  ← Emotion tag (if set)
│ ⚙ SetVariable: HasMet = true               │  ← SideEffect pill
└────────────────────────────────────────────┘
      ●                                    ●
    Input                               Output
```

- **Title bar** carries the speaker color configured in the asset's Speakers panel.
- **Inline text** shows the first 2–3 lines. Double-clicking the text opens an inline edit field — no Details panel needed.
- **Emotion tags** appear as compact chips below the text when they are set.
- **SideEffect pills** show inline actions (e.g. `SetVariable`, `AddTag`) as single-line entries at the bottom edge of the node.

> 📸 **Image placeholder:** `sayline-node-annotated.png` — A single SayLine node with annotation overlays.
> *Setup:* Asset editor, a single SayLine node selected and centered. Speaker: Guard (dark red title bar). Text: "Halt! Who are you?". Emotion tag `Dialogue.Emotion.Stern` visible as a small blue chip. SideEffect pill `SetVariable HasMet = true` below it. Red arrows with labels pointing at: title bar ("Speaker Color"), text ("Inline Text"), tag chip ("EmotionTag"), pill ("SideEffect").

## Anatomy: PlayerChoice node

```
┌────────────────────────────────────────────┐
│ [purple] Player Choice                     │
├────────────────────────────────────────────┤
│ You answer:                                │  ← PromptText
│                                            │
│ 🔓 A friend of the king.            ─────► │  ← Choice 0 (available)
│ 🔒 I know the password.             ─────► │  ← Choice 1 (blocked, hidden)
│ ⚠️ That's none of your business.    ─────► │  ← Choice 2 (visible but locked)
└────────────────────────────────────────────┘
      ●
    Input
```

Each choice has its own **output pin**. The structure of the decision is the structure of the graph — you can see immediately how many paths there are.

| Icon | Meaning in runtime preview |
| --- | --- |
| 🔓 Open lock | Selectable — requirement met or no requirement set. |
| ⚠️ Yellow warning symbol | Visible but not selectable. Tooltip shows the reason. |
| 🔒 Closed lock | Invisible to the player. Filtered out. |

> 📸 **Image placeholder:** `playerchoice-node-preview.png` — PlayerChoice node in preview mode with lock icons.
> *Setup:* Preview Runner active. The PlayerChoice node is centered. Choice 0 shows a green open lock (no requirement). Choice 1 shows a red lock (requirement failed, `FailedAndHidden`). Choice 2 shows a yellow warning triangle (requirement failed, `FailedButVisible`, greyed out). Output pins for choices 0 and 2 visible, output for choice 1 dashed.

## Branch vs. PlayerChoice

Both nodes evaluate conditions — but with different intent:

| | Branch | PlayerChoice |
| --- | --- | --- |
| Who decides? | The logic (automatically) | The player (interactively) |
| Appearance | Compact diamond box | Wide box with choice pills |
| Pins | True / False / Fallback | One pin per choice |
| Typical question | "Which path does the dialogue take?" | "What does the player answer?" |

> 📸 **Image placeholder:** `branch-vs-playerchoice.png` — Both node types side by side.
> *Setup:* Two nodes side by side. Left: `Branch` node (blue title bar, diamond shape, three pins: True, False, Fallback, requirement pill in body). Right: `PlayerChoice` node (purple, wide box, three choice pills each with one output pin). Red arrows with labels: "Logic decides" (Branch), "Player decides" (PlayerChoice).

## Special forms: Entry, Exit, SubGraph, Knot

| Node | Shape | Function |
| --- | --- | --- |
| **Entry** | Green capsule, output pin only | Start point of the dialogue. Exactly one per asset. Cannot be deleted. |
| **Exit** | Red capsule, input pin only | End point. Status: Completed or Failed. |
| **SubGraph** | Box with folder icon | Double-click opens the embedded sub-graph with a breadcrumb path at the top. |
| **Knot** | Small circle | Wire routing. Resolved at compile time — the runtime sees only the direct connection. |

## Color reference

| Node type | Default color |
| --- | --- |
| Entry | Green |
| Exit | Red |
| SayLine | Per-speaker (from Speakers panel) |
| PlayerChoice | Purple |
| Branch | Blue |
| Random Line | Yellow |
| Wait | Teal |
| Link / SubGraph | Orange |
| SetVariable / Event | Violet / Gold |

All colors can be customized under **Project Settings → MayDialogue Editor**.

> 📸 **Image placeholder:** `color-reference-nodes.png` — A row of different node types with color chips.
> *Setup:* All node types side by side in a row, each empty but colored. Left to right: Entry (green), Exit (red), SayLine (red/orange, speaker color), PlayerChoice (purple), Branch (blue), RandomLine (yellow), Wait (teal), Link (orange), SetVariable (violet). No connections between nodes.

## Comments and auto-layout

- **Comment boxes** (key `C` or right-click → Add Comment): Colored border around a group of nodes. Moves with the group when you drag it.
- **Auto-Layout button** in the toolbar: Sorts the graph by depth (Entry = left, Exits = right). Useful for a graph that has grown chaotic. A manual fine-tuning pass is worth doing afterwards.

## The Outline panel

When the graph gets too large to search for a specific line: the **Outline panel** lists all nodes in a flat list — with speaker color chip, text preview, and type badge. Full-text search and type filter are built in. Clicking an entry jumps directly to the node in the graph.

> 📸 **Image placeholder:** `outline-panel.png` — Outline panel with a populated node list.
> *Setup:* Outline panel tab open (to the right of the graph). List shows 8–10 entries: each entry has a colored chip on the left (speaker color), then speaker name and first line of text (truncated to 60 characters), and a type badge on the right (`Say`, `PC`, `Br`, `X` etc.). Search field at top is empty. One entry is highlighted blue (selected).

## Summary

The graph is designed to be **read** without clicking every node: speaker colors provide orientation, inline text shows content, Sub-Node pills show conditions and actions, and shapes distinguish semantics. The Outline panel complements the graph as a linear text list.

Next: [Instance & Lifecycle](instance-lifecycle.md).
