# Core Nodes

The nine Core Nodes are the backbone of every dialogue. They determine where the dialogue starts, who speaks, where it branches, and how it ends.

| Node | Purpose | Advance Mode |
| --- | --- | --- |
| [Entry](entry.md) | Graph start point | Immediate (automatic) |
| [Exit](exit.md) | End point with status | — (ends dialogue) |
| [Say Line](say-line.md) | Speaker says a line | Manual / Timer / AfterVoice / AfterAnimation / Immediate |
| [Player Choice](player-choice.md) | Player selects an option | — (waits for selection) |
| [Branch](branch.md) | Automatic conditional branching | Immediate |
| [Random Line](random-line.md) | Random line from multiple variants | like Say Line |
| [Wait](wait.md) | Pause on time, event, or condition | on trigger |
| [Link](link.md) | Switch to another asset | Immediate / like target dialogue |
| [SubGraph](sub-graph.md) | Expand an internal sub-graph | Immediate |

> 📸 **Image placeholder:** `core-overview-graph.png` — Overview of all Core Nodes in a demo graph.
> *Setup:* New test asset with the following Nodes from left to right: `Entry` (green) → `SayLine` → `PlayerChoice` with two Choices → one `SayLine` each → `Exit Completed` (top) and `Exit Failed` (bottom). Separately alongside: `Branch`, `Wait`, `RandomLine`, `Link`, and `SubGraph` Nodes side by side, each unconnected, for a visual overview.

Each page documents properties, runtime behavior, example graphs, and common pitfalls.
