---
description: Show a PlayerChoice option only when the player has a specific GameplayTag.
---

# Choice Visible Only with Tag

## Scenario

A merchant has three purchase options. A fourth option — *"I'm a Guild member, give me a discount."* — should only appear when the player has the tag `Guild.Member.Active`. Non-members don't see the option at all; they don't guess it exists. This pattern is the classic technique for hidden dialogue paths.

## What You Will Learn

- Attach a HasTag Requirement directly to a Choice (not to a Branch).
- Understand the difference between `FailedAndHidden` and `FailedButVisible`.
- Combine multiple Requirements on a Choice (AND logic).
- Set Choice tags for external systems.

## Prerequisites

- [Branching with Conditions](branching-conditions.md) completed.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: "What can I offer you?"]
   │
   ▼
[PlayerChoice]
   ├─ "Buy a health potion."        (no Req) → [SayLine: "10 gold."] → [Exit]
   ├─ "Look at the armor."          (no Req) → [SayLine: "Right this way."] → [Exit]
   ├─ "I'm just browsing."          (no Req) → [Exit]
   └─ "Guild discount, please."     (Req: HasTag Guild.Member.Active, FailedAndHidden)
         └─► [SayLine: "Of course! 20% for guild members."] → [Exit]
```

> 📸 **Image placeholder:** `choice-with-tag-requirement-graph-overview.png` — Overview graph of the merchant dialogue.
> *Setup:* Asset `DA_Merchant_Shop` open. SayLine → PlayerChoice with four Choice sub-nodes in the body. Last Choice has a lock icon next to the text. In the Preview Runner (without tag), only three choices are visible.

## Step by Step

### 1. Create the PlayerChoice

Asset: `DA_Merchant_Shop`. SayLine *"What can I offer you?"* → **PlayerChoice** node.

In the Details panel of the PlayerChoice: create four choices via **Add Choice**.

### 2. Fill in the First Three Choices Normally

For each of the open choices: enter `ChoiceText`, wire up the output pin.

### 3. Fourth Choice with Requirement

Choice 4: `ChoiceText = "Guild discount, please."`. Under **Requirements → Add → HasTag**:

| Property | Value |
|----------|------|
| `RequiredTag` | `Guild.Member.Active` |
| `bCheckOnInstigator` | `true` |
| `FailureResult` | `FailedAndHidden` |

`FailedAndHidden` = the choice doesn't appear in the list at all for non-members.

> 📸 **Image placeholder:** `choice-with-tag-requirement-choice-details.png` — Details panel of Choice 4 with HasTag Requirement.
> *Setup:* PlayerChoice node selected, Choice 4 expanded in the Details panel. `ChoiceText = "Guild discount, please."`, below it Requirements: `HasTag: RequiredTag = Guild.Member.Active, bCheckOnInstigator = true, FailureResult = FailedAndHidden`.

### 4. Wire Up the Output

Choice 4 output pin → SayLine *"Of course! 20% for guild members."* → Exit.

### 5. Compile and Test

Preview Runner: without the tag you see three choices. Add the tag to the Player ASC in the Debugger → the fourth choice appears.

## FailedAndHidden vs. FailedButVisible

| Setting | Behavior | When to use |
|-------------|-----------|-------------|
| `FailedAndHidden` | Choice invisible | Player shouldn't know the option exists |
| `FailedButVisible` | Choice visible but grayed out + tooltip | Player should see what they haven't unlocked yet |

The tooltip message comes from `UnavailableReason` — leave it empty if no explanation is desired.

## Combining Multiple Requirements

You can stack multiple Requirements on a Choice. Evaluation is **AND**: all must return `Passed`.

Example: guild member AND enough gold:

```text
Requirements:
  [0] HasTag(Guild.Member.Active)   FailedAndHidden
  [1] CheckAttribute(Gold >= 50)    FailedButVisible  UnavailableReason: "Not enough gold."
```

Result: non-members don't see the option. Members without enough gold see it grayed out with a hint.

## Blueprint Triggering

No special code — standard dialogue start:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Merchant_Shop
   └─ ...
```

> 📸 **Image placeholder:** `choice-with-tag-requirement-ingame.png` — Preview Runner with three visible choices (tag not set).
> *Setup:* Preview Runner open. Choice list shows three entries. Fourth choice not visible. Alongside: add tag to Player ASC → restart Preview → four choices visible.

## Variations / Going Further

- The same pattern with an **attribute instead of a tag** → [Choice with Attribute Condition](choice-with-attribute-requirement.md).
- Set Choice tags: fill in `ChoiceTags` on the Choice sub-node. External systems (quest system) can react to "player selected Choice with tag X".
- `FailedButVisible` for a classic RPG pattern: always show the Charisma option, only selectable if Charisma ≥ 5.

## Troubleshooting

**Choice always invisible, even though the tag is set.**
`bCheckOnInstigator = false` → tag is checked on the NPC instead of the player. Set to `true`.

**Choice always visible, even though the tag is missing.**
`FailureResult = FailedButVisible` instead of `FailedAndHidden`. Or `RequiredTag` has a typo — the Requirement matches nothing and returns `Passed`.

**Multiple Requirements, choice never selectable.**
AND logic: all Requirements must return `Passed`. Check each Requirement status individually in the Debugger.
