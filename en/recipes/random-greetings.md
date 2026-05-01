---
description: RandomLine node for fresh NPC greetings every conversation – with weighting and no-repeat.
---

# Random Greetings

## Scenario

A merchant says a different greeting on every interaction, drawn from a pool of five lines. One of them should appear rarely and only show up when the merchant hasn't greeted the player just moments ago. Without this recipe, every dialogue would sound identical — with it, the world feels alive.

## What You Will Learn

- Create a RandomLine node and fill it with multiple entries.
- Use the weight system to favor rare lines less often.
- Enable `bAllowRepeat = false` for automatic no-repeat behavior.
- Use a Requirement on an entry for more complex filtering.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.

## Mini-Graph

```text
[Entry]
   │
   ▼
[RandomLine: Merchant]
   ├─ "Hello, traveler."            (Weight 3)
   ├─ "Good to see you!"           (Weight 3)
   ├─ "Welcome back."              (Weight 3)
   ├─ "How are you?"               (Weight 3)
   └─ "I haven't seen you in..."   (Weight 1, Req: LastGreetingRecent=false)
   │
   ▼
[SayLine: "What are you looking for today?"]
   │
   ▼
[PlayerChoice: Purchase options ...]
```

> 📸 **Image placeholder:** `random-greetings-graph-overview.png` — Overview graph of the merchant dialogue with RandomLine.
> *Setup:* Asset `DA_Merchant_Greet` open. Left to right: `Entry` → `RandomLine` node (dice icon in title, all 5 entries visible as pills in the body) → `SayLine "What are you looking for"` → `PlayerChoice`. Connections clear, entries with Weight values visible in the node body.

## Step by Step

### 1. Create the Asset and Speaker

Asset: `DA_Merchant_Greet`. Speaker: `DisplayName = Merchant`, `SpeakerTag = Dialogue.Speaker.Merchant`.

### 2. Insert the RandomLine

From the Entry output → **Create Node → Random Line**. The node displays a dice icon and an empty entry list.

### 3. Fill in the Entries

In the Details panel under `LineEntries`, click **Add Entry** four times and fill them in:

| Text | Weight |
|------|--------|
| "Hello, traveler." | 3 |
| "Good to see you!" | 3 |
| "Welcome back." | 3 |
| "How are you?" | 3 |

Then **Add Entry** for the fifth entry: *"I haven't seen you in ages."*, Weight = 1.

> 📸 **Image placeholder:** `random-greetings-randomline-details.png` — Details panel of the RandomLine node with all entries.
> *Setup:* RandomLine selected. Details panel shows: `LineEntries` with 5 entries, each with text and weight. Last entry Weight = 1, below it Requirements expanded with `CheckParticipantVariable: LastGreetingRecent = false`.

### 4. Guard the Rare Line

On the fifth entry, under **Requirements → Add → CheckParticipantVariable**:

| Property | Value |
|----------|------|
| `VariableName` | `LastGreetingRecent` |
| `ExpectedValue` | `false` |
| `FailureResult` | `FailedAndHidden` |

When the variable is `true` (just spoken), the entry is completely hidden.

### 5. Enable No-Repeat

On the RandomLine node: `bAllowRepeat = false`. The plugin remembers the last drawn index per participant and picks a different one next time.

### 6. Set the Recent Flag (optional)

On the Entry node, attach a **SideEffect → SetVariable**:
- `Variable`: `LastGreetingRecent`, Scope: `Participant`, Type: `Bool`, Value: `true`.

On the Exit node, the same SideEffect with Value `false` — or reset it via a timer in the Participant Blueprint after 60 seconds.

### 7. Wire Up the Continuation

RandomLine output → SayLine *"What are you looking for today?"* → PlayerChoice (or further dialogue block).

## Weighting Formula

```text
P(Entry_i) = Weight_i / Σ Weight_j    (visible entries only)
```

For our example (all visible): rare line = `1 / (3+3+3+3+1) = 1/13 ≈ 7.7 %`.

When the rare line is filtered out by its Requirement, the system only calculates with the remaining four entries — the total weight adjusts automatically.

## Blueprint Triggering

No special code needed — standard `Start Dialogue` call:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Merchant_Greet
   ├─ Instigator: Get Player Pawn
   └─ Target:     Self
```

> 📸 **Image placeholder:** `random-greetings-bp-trigger.png` — Blueprint trigger on the merchant.
> *Setup:* Merchant actor BP. `Event OnInteract` → `Start Dialogue`, pins filled.

## Variations / Going Further

- Use a RandomLine **in the middle** of a dialogue, not just as an opener — it works anywhere a SayLine would fit.
- Control Weight dynamically via a variable: make rare lines more common after certain quest milestones.
- Extract the RandomLine block as a fragment and reference it via Link from multiple NPC assets → [Reusable Dialogue Fragments](linking-dialogues.md).

## Troubleshooting

**The same entry is always drawn.**
The Preview Runner uses a deterministic seed — PIE will look different. Also: if `bAllowRepeat = false` and only one entry is visible (all others filtered by Requirements), there's no choice.

**The rare line keeps appearing despite Weight 1.**
All other entries are filtered out by Requirements. Check in the Debugger which entries are visible. Often: a typo in the variable name of the Requirement check.

**RandomLine never takes its output.**
Check if `bAdvanceAutomatically` is enabled on the node. If the inner entries have `AdvanceMode = Manual`, the node waits for an Advance call that never comes.
