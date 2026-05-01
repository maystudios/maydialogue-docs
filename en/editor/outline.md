---
description: All nodes as a searchable list — filter, search, and jump directly by clicking.
---

# Outline

The Outline tab shows all nodes in your dialogue as a **sorted list** — with speaker chips, text previews, and type badges. Clicking a row jumps directly to the node in the graph.

> 📸 **Image placeholder:** `outline-overview.png` — Outline tab with seven entries of a complete dialogue.
> *Setup:* Outline tab active. Seven rows: Entry (no chip, badge "E"), SayLine Guard "Halt! Who are you?" (dark red chip, badge "Say"), PlayerChoice "You answer:" (blue chip, badge "PC"), SayLine Guard "Then pass in peace." (dark red chip), SayLine Guard "Then get out!" (dark red chip), Exit Completed (no chip, badge "X"), Exit Failed (no chip, badge "X"). Search field at top empty, filter set to "All".

## Why do you need the Outline?

The graph shows nodes spatially — you search for them by their position. Once an asset has more than 20 nodes, spatial searching becomes tedious. The Outline answers questions like:

- *"Who says that again?"* → speaker chip immediately visible
- *"How many PlayerChoices does this dialogue have?"* → filter: Player Choice
- *"Which SayLine talks about the cellar?"* → search field: "cellar"

## Anatomy of a row

| Element | Meaning |
| --- | --- |
| **Color chip** | Speaker color (grey for nodes with no speaker) |
| **Primary text** | Speaker display name or node title |
| **Secondary text** | Dialogue text preview (truncated to 60 characters) |
| **Type badge** | Two-letter abbreviation on the right |

### Type badge reference

| Badge | Node type |
| --- | --- |
| `E` | Entry |
| `X` | Exit |
| `Say` | SayLine |
| `PC` | PlayerChoice |
| `Br` | Branch |
| `Rn` | RandomLine |
| `Wt` | Wait |
| `Ln` | Link |
| `Sg` | SubGraph |
| `Ev` | Fire Event |
| `Fx` | Apply Effect |
| `SV` | Set Variable |
| `PS` | Play Sound |
| `CF` | Camera Focus |
| `CS` | Camera Shake |
| `PA` | Play Animation |

## Searching

Search field at the top edge of the Outline tab. The list filters **live** as you type.

Searched content:
- Speaker display name and node title (primary text)
- Dialogue text preview (secondary text)
- Type badge abbreviations (e.g. "Say" shows all SayLines)

> 📸 **Image placeholder:** `outline-search.png` — Outline with "cellar" in the search field, two filtered results visible.
> *Setup:* Outline tab, "cellar" typed in search field. Only two SayLines remaining: "You know what's in the cellar." and "The cellar is sealed." Both with a dark red Guard chip. Red arrow on the search field.

## Filtering by node type

The dropdown to the right of the search field selects a type filter:

| Filter | Shows |
| --- | --- |
| **All** | All nodes (default) |
| **Say Line** | Only SayLines |
| **Player Choice** | Only PlayerChoice nodes |
| **Branch** | Only Branch nodes |
| **Actions** | Set Variable, Fire Event, Apply Effect, Play Sound, camera and animation nodes |
| **Flow Control** | Wait, Link, SubGraph |
| **Special** | Entry, Exit, Knot |

Filter and search field can be combined: filter "Say Line" + search text "Guard" shows all SayLines for the Guard.

> 📸 **Image placeholder:** `outline-filter-dropdown.png` — Filter dropdown open, "Player Choice" highlighted, two PC nodes visible in the list below.
> *Setup:* Outline tab, filter dropdown expanded. "Player Choice" highlighted. In the background of the list two PlayerChoice entries with blue chips. Red arrow on the dropdown.

## Click-to-Jump

Clicking an Outline row:
1. Selects the node in the graph.
2. Centers the graph camera on the node.

This lets you find any node in a 150-node asset in under a second — without manually scrolling and searching in the graph.

## Live update

The Outline updates automatically when you add, delete, or rename nodes in the graph. You don't need to reopen the tab or manually refresh.

{% hint style="info" %}
In rare cases where a graph-change signal doesn't arrive immediately, the Outline has an automatic fallback sync (hash check in the background). The list stays in sync regardless.
{% endhint %}

## Typical use cases

**"Who says that?"**
Enter text in the search field. The first match shows the speaker chip and name.

**"How many choices does this dialogue have?"**
Filter by Player Choice → count the rows.

**"Check all camera nodes"**
Filter by Actions → only camera and animation nodes visible.

**"Jump to a specific Exit"**
Filter by Special → both Exits visible, click the right one.

> 📸 **Image placeholder:** `outline-click-to-jump.png` — Outline with a row selected, in the background the graph jumps to the corresponding node.
> *Setup:* Outline tab, second SayLine row clicked (highlighted blue). In the graph area behind it the camera centers on this SayLine, node slightly highlighted (selection color). Red arrow on the selected row in the Outline.

Next: [Find-in-Dialogue →](find.md)
