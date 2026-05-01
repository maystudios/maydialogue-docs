---
description: Set breakpoints, use Step Over/Into/Out, and watch variables live in the Watch panel.
---

# Debugger & Breakpoints

The PIE debugger is your tool when a dialogue is not behaving as expected in the actual game. It works just like the Blueprint debugger — but for dialogues.

Use the [Preview Runner](preview-runner.md) for quick text iteration. Use the debugger when you need real GAS state, quest state, or NPC behaviour in the loop.

## Setup in three steps

1. Open the dialogue in the Asset Editor.
2. Set breakpoints on relevant nodes (right-click → **Toggle Breakpoint** or `F9`).
3. **Start PIE** and trigger the dialogue in the game.

As soon as execution reaches a node with a breakpoint:
- The dialogue pauses.
- The editor window comes to the foreground.
- The graph centres on the paused node.
- The node lights up in the **active debug colour** (default: yellow).

> 📸 **Image placeholder:** `debugger-breakpoint-hit.png` — Graph with a paused Branch node, highlighted in yellow. Watch panel visible at the bottom.
> *Setup:* PIE running, dialogue triggered. Branch node "CheckDialogueVariable(AngerLevel >= 3)" with breakpoint. Node lights up yellow (ActiveDebugColor). Red arrow pointing to the glowing node. DebuggerWatch tab visible in the background with `AngerLevel = 2`.

## Managing breakpoints

### Setting and removing

| Method | Effect |
| --- | --- |
| `F9` (node selected) | Toggle breakpoint |
| Right-click on node → Toggle Breakpoint | Toggle breakpoint |

An active breakpoint appears as a **red dot in the top-right corner of the node**.

### Disabling (without deleting)

Right-click on the breakpoint indicator → **Disable**. The breakpoint is preserved but will not trigger. Useful when you want to run through a section temporarily without interruption.

### Disabling all breakpoints

**Debug menu → Disable All Breakpoints**. All breakpoints in the asset are disabled but not deleted. Re-enable them with **Enable All Breakpoints**.

{% hint style="info" %}
Breakpoints are saved **per user** in the project and survive editor restarts. You do not need to re-set them after every restart.
{% endhint %}

## Step controls

After a breakpoint hit, the following actions are available:

| Action | Shortcut | Effect |
| --- | --- | --- |
| **Continue** | `F5` | Continue until the next breakpoint |
| **Step Over** | `F10` | Execute the current node, pause at the next one |
| **Step Into** | `F11` | For link/sub-graph nodes: jump into the linked graph |
| **Step Out** | `Shift+F11` | Jump out of the current sub-graph/link scope |

> 📸 **Image placeholder:** `debugger-step-controls.png` — Toolbar or debug menu with step buttons annotated.
> *Setup:* PIE debugger active (breakpoint hit). Debug toolbar shows Continue (F5), Step Over (F10), Step Into (F11), Step Out (Shift+F11). Red arrows with labels on each button.

## Active highlighting in the graph

During a debug session:

| Colour | Meaning |
| --- | --- |
| **Yellow** (ActiveDebugColor) | Currently paused node |
| **Pale blue** (HistoryDebugColor) | Nodes already visited in this session |

This lets you see at a glance which path the dialogue has taken.

> 📸 **Image placeholder:** `debugger-history-highlight.png` — Graph with the paused node (yellow) and three previously visited nodes (pale blue).
> *Setup:* Debug session: Entry (pale blue), SayLine Guard (pale blue), PlayerChoice (pale blue), Branch (yellow, currently paused). Dialogue path clearly visible. Legend annotation in the image: yellow = current, pale blue = visited.

## Watch panel

The **DebuggerWatch** tab (at the bottom) shows during the pause:

- **Dialogue variables** of the active instance with their current value
- **Participant variables** of all participating actors
- Data type and string representation of the value

You can **edit values directly in the Watch panel**. Change a value, press `F5` (Continue) — execution continues with the new value.

> 📸 **Image placeholder:** `debugger-watch-panel.png` — DebuggerWatch tab with four variables, one currently being edited.
> *Setup:* DebuggerWatch tab active at the bottom. Four rows: `HasAskedName` (Bool, false), `AngerLevel` (Int, 2 — currently in edit mode, input field open with cursor), `HasMet` (Bool, true, Participant scope), `Friendship` (Float, 0.7, Participant scope). Red arrow pointing to the editable `AngerLevel` field.

## Typical debug workflows

### "Why is Choice 2 not showing?"

1. Set a breakpoint on the PlayerChoice node.
2. Start PIE, trigger the dialogue.
3. On hit: check the Watch panel — which tag or variable is missing?
4. Set the value to the expected one in the Watch panel, press `F5`.
5. Check whether the choice now appears.

### "Which output does the Branch take?"

1. Set a breakpoint on the Branch node.
2. On hit: `F10` (Step Over).
3. The next highlighted node shows the chosen exit.

### "What happens after the FireEvent?"

1. Set a breakpoint on the FireEvent node.
2. On hit: note the Watch panel, `F10` (Step Over).
3. Observe whether the following Wait node wakes up.

### "Is the SubGraph running correctly?"

1. Set a breakpoint on the SubGraph node.
2. On hit: `F11` (Step Into) — you jump into the sub-graph.
3. Continue stepping there.
4. `Shift+F11` (Step Out) to return to the parent graph.

## Combining the debugger and Preview

The two tools complement each other perfectly:

| Phase | Tool |
| --- | --- |
| Writing and testing text, choices, and variables | Preview Runner |
| Validating GAS tags, quest state, and real NPC logic | PIE Debugger |

Typical workflow: 90% of the work in the Preview Runner, final validation in the debugger.

## Known limitations

- **Cross-asset Step Into:** When a Link node jumps to another asset, the editor does not open the target asset automatically. Workaround: open the target asset manually and set a breakpoint there. You can find the link target in the Details panel of the Link node in the source asset (property `TargetAsset`).
- **Async nodes:** Wait and PlayAnimation nodes only pause when they have finished — not midway through.

Next: [Comfort Features →](comfort-features.md)
