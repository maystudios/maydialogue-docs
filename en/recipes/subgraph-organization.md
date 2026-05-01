---
description: Split a large dialogue into manageable sections with SubGraph nodes – without splitting assets.
---

# SubGraph Organization

## Scenario

Your main quest NPC has a long dialogue with an intro, two conversation topics, and a closing block. The graph grows to 30+ nodes and becomes hard to read. With SubGraphs you collapse each block into a single box — the main graph stays readable like a table of contents.

## What You Will Learn

- Create and fill a SubGraph node.
- Navigate between main graph and sub-graph using breadcrumb navigation.
- SubGraph scope: variables and speakers are inherited from the parent.
- The difference between SubGraph (internal) and Link (external).

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- Understanding of the [Link node](linking-dialogues.md).

## Mini-Graph

```text
[Entry]
   │
   ▼
[SubGraph: "Intro"]
   │  (internally: SayLine × 2 → Exit)
   ▼
[SubGraph: "Topic A: The Artifact"]
   │  (internally: SayLine → PlayerChoice → Branch → ...)
   ▼
[SubGraph: "Topic B: The Threat"]
   │  (internally: Branch → SayLine × 3 → ...)
   ▼
[SubGraph: "Closing"]
   │  (internally: SayLine → AddTag → Exit)
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `subgraph-organization-main-graph.png` — Main graph with four SubGraph boxes.
> *Setup:* Asset `DA_QuestNPC_Main` open. Visible: Entry → four SubGraph nodes in sequence (folder icon in title bar, names readable) → Exit. Each box compact, no internal nodes visible. The graph fits entirely in the viewport.

## Step by Step

### 1. Create the Asset with Many Nodes

Asset: `DA_QuestNPC_Main`. First lay out all 30 nodes flat in the graph and test them.

### 2. Create the First SubGraph

Right-click in the graph → **Create Node → SubGraph**. Name in the title: *"Intro"*. The SubGraph node appears as a single box with a folder icon.

### 3. Move Nodes into the SubGraph

Double-click on the SubGraph node → a new tab opens with its own Entry + Exit. Copy the intro nodes from the main graph via **Ctrl+C / Ctrl+V** into it. Connect them to the SubGraph's internal Entry and Exit. Delete the original nodes from the main graph.

> 📸 **Image placeholder:** `subgraph-organization-subgraph-inside.png` — Opened SubGraph "Intro" with internal nodes.
> *Setup:* Tab "Intro" at the top with breadcrumb `DA_QuestNPC_Main > Intro`. In the graph: SubGraph Entry (small green node) → two SayLines → SubGraph Exit. Breadcrumb bar visible at the top edge of the graph panel.

### 4. Breadcrumb Navigation

At the top of the graph panel you'll see the breadcrumb bar: `DA_QuestNPC_Main > Intro`. Click `DA_QuestNPC_Main` to return to the main graph.

### 5. Extract the Remaining Blocks

Repeat steps 2–3 for *Topic A*, *Topic B*, and *Closing*. Connect the SubGraph nodes in the main graph in the correct order.

### 6. Name the SubGraphs

Every SubGraph has a `Description` field in the Details panel. Use precise names: *"Intro – Greeting"*, *"Topic A – Artifact Question"*. These names also appear in the Outline panel.

> 📸 **Image placeholder:** `subgraph-organization-outline-panel.png` — Outline panel with expanded SubGraph entries.
> *Setup:* Outline panel open on the left. Entries: SubGraph "Intro" with child nodes below (SayLine × 2), SubGraph "Topic A" with children, etc. Speaker color chips visible.

## SubGraph vs. Link – Quick Comparison

| | SubGraph | Link |
|--|----------|------|
| Lives in | Same asset | Its own asset |
| Reusable | No | Yes |
| Speaker scope | Inherited from parent | Own scope |
| For | Internal structure | Shared fragments |

## Blueprint Triggering

No special code. SubGraphs are internal — from the outside the dialogue starts normally:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_QuestNPC_Main
   └─ ...
```

## Variations / Going Further

- **Conditionally skip a SubGraph**: instead of a direct connection, put a Branch before it — skip a SubGraph depending on quest status.
- For true reusability, extract the SubGraph into its own asset and reference it via [Link](linking-dialogues.md).
- In the Outline panel, use **Filter by SubGraph** to quickly navigate between sections.

## Troubleshooting

**SubGraph node has no output connection.**
The internal SubGraph Exit must be connected to the SubGraph's internal Exit node — not to the main Exit node. Double-click → check the internal Exit.

**Variables visible in SubGraph, but values are wrong.**
SubGraph inherits the parent asset's scope. If you set a variable in the SubGraph and read it later in the main graph, it's the same scope — it works correctly. Check for typos in the variable name.

**Breadcrumb shows wrong depth.**
Nested SubGraphs are possible — the breadcrumb shows the full depth. Navigate up by clicking any breadcrumb entry.
