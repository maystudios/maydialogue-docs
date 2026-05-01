---
description: The MayDialogue asset editor — which panels exist and how to work with them.
---

# The Editor

When you double-click a MayDialogue asset in the Content Browser, the **asset editor** opens. Here you design, test, and debug dialogues — without writing a single line of code.

> 📸 **Image placeholder:** `editor-overview-annotated.png` — Full view of the asset editor with an example dialogue open.
> *Setup:* Open asset `DA_Guard_Greeting`. All standard tabs visible: graph large in the center, Details panel on the right, Outline on the left, Compiler Results at the bottom. Red arrows annotate: 1) toolbar with Save/Compile/Play buttons, 2) graph canvas with three connected nodes, 3) Speakers panel tab in the upper right.

## Panel overview

| Panel | Purpose | Page |
| --- | --- | --- |
| **Graph** | Build dialogue structure — connect nodes, write text | [Graph Panel](graph-panel.md) |
| **Details** | Edit properties of the selected node | [Asset Editor](asset-editor.md) |
| **Speakers** | Create speakers, set colors and portraits | [Speakers Panel](speakers-panel.md) |
| **Variables** | Declare dialogue variables and set default values | [Variables Panel](variables-panel.md) |
| **Outline** | See all nodes as a searchable list, jump to any by clicking | [Outline](outline.md) |
| **Find Results** | Search for text, tags, and variable names across the entire asset | [Find-in-Dialogue](find.md) |
| **Preview** | Play the dialogue directly in the editor — no PIE needed | [Preview Runner](preview-runner.md) |
| **Debugger Watch** | Watch variables live while PIE is running | [Debugger](debugger.md) |
| **Compiler Results** | Errors and warnings after the compile pass | [Asset Editor](asset-editor.md) |
| **Palette** | Insert available node types via drag and drop | [Graph Panel](graph-panel.md) |

All tabs are freely dockable. Your layout is saved per user.

## Default layout

```
┌──────────────────────────────────────────────────────────────────┐
│  [Save]  [Compile]  [Auto-Layout]  [Play]  [Find]  [Breakpoint]  │  ← Toolbar
├───────────┬──────────────────────────────────────┬───────────────┤
│           │                                      │               │
│  Outline  │           Graph Canvas               │    Details    │
│           │                                      │    Speakers   │
│           │                                      │    Variables  │
│           │                                      │    Palette    │
├───────────┴──────────────────────────────────────┴───────────────┤
│        Compiler Results  │  Find Results  │  Debugger Watch      │
└──────────────────────────────────────────────────────────────────┘
```

> 📸 **Image placeholder:** `editor-default-layout.png` — Default tab layout with a new empty asset.
> *Setup:* Create and open a new `DA_Empty` asset. Only the Entry node visible in the graph. Tabs in the sidebar show Details, Speakers, Variables as tab headers.

## Recommended workflow

For a new dialogue from scratch:

1. **Create the asset** – Right-click in the Content Browser → Miscellaneous → MayDialogue Asset.
2. **Define speakers** – Open the Speakers panel, add speakers, set color and DisplayName.
3. **Declare variables** – Variables panel, if the dialogue needs to store state.
4. **Build the graph** – Place nodes, connect them, enter text inline.
5. **Compile** – Press `F7`, check Compiler Results for errors.
6. **Test with Preview Runner** – Click Play, run through the dialogue without PIE.
7. **PIE Debugger** – For final validation with real GAS state in the running game.

> 📸 **Image placeholder:** `editor-workflow-steps.png` — Toolbar area with numbered red arrows on the Compile button (step 5) and Play button (step 6).
> *Setup:* Any asset with two SayLine nodes open. Toolbar in focus.

## Which tool for which task

| Question / task | Tool |
| --- | --- |
| Build dialogue flow | Graph panel |
| Maintain speaker colors and portraits | Speakers panel |
| Quick test of a new line | Preview Runner |
| "Where is sentence X?" | Find-in-Dialogue |
| "How many choices does the dialogue have?" | Outline (filter: Player Choice) |
| Track down a bug with real quest state | PIE Debugger |
| Clean up a messy graph | Auto-Layout + Comment Boxes |

> 📸 **Image placeholder:** `editor-panel-closeup-speakers.png` — Speakers panel expanded with two entries (Guard, Player), color chips visible.
> *Setup:* Speakers tab active, two speakers `Dialogue.Speaker.Guard` (dark red) and `Dialogue.Speaker.Player` (grey) entered, DisplayNames and portrait slots visible.

> 📸 **Image placeholder:** `editor-panel-closeup-outline.png` — Outline tab with four entries: Entry, SayLine Guard, PlayerChoice, Exit.
> *Setup:* Small asset with Entry → SayLine → PlayerChoice → Exit. Outline shows color chips, text previews, and type badges.

Let's go: [Asset Editor →](asset-editor.md)
