---
description: Dialogue scope vs. participant scope — when to use which and how to access them.
---

# Variables & Scopes

MayDialogue has two places where variables live. Choosing the right scope is the only decision you need to make — the rest is mechanically identical.

## The two scopes

| | Dialogue scope | Participant scope |
| --- | --- | --- |
| **Lives** | Only during the conversation | Survives the conversation |
| **Stored in** | The running instance | The participant component (SaveGame-flagged) |
| **Typical use** | Intermediate states, counters, aggregations within a scene | "Has met me before", relationship values, world knowledge |

Rule of thumb: if the variable is irrelevant after the dialogue → **Dialogue**. If it should still apply in the next conversation → **Participant**.

## Decision tree

```text
Need a new variable?
    │
    ├─ Only relevant for this conversation?
    │       └─→ Dialogue scope
    │
    └─ Must it survive the conversation?
            ├─ Until a level change is enough?
            │       └─→ Participant scope (not persisted)
            └─ Must it stay in the save game?
                    └─→ Participant scope (SaveGame automatic)
```

**Examples:**

| Variable | Scope | Why |
| --- | --- | --- |
| "Has the player already asked for the name" | Dialogue | Only relevant in this session |
| "How many times was the player provoked in this scene" | Dialogue | Counter, conversation ends after |
| "Player knows the secret" | Participant + Save | Applies to all later conversations |
| "Friendship level with NPC" | Participant + Save | Persistent relationship |
| "Has spoken in this level already" | Participant, not persisted | Cross-conversation but level-scoped |

## Declaring variables

Variables are declared in the **Variables panel** of the asset editor — not in Blueprint, not in C++.

> 📸 **Image placeholder:** `variables-panel.png` — Variables panel with several entries.
> *Setup:* Asset editor, Variables panel tab open. Table with four columns: Name, Type, Scope, Default. Three entries visible: 1. `HasAngered | Bool | Dialogue | false`. 2. `ProvocationCount | Int | Dialogue | 0`. 3. `HasMet | Bool | Participant | false`. Add button visible at bottom left.

Supported types: `Bool`, `Int`, `Float`, `String`, `Tag` (FGameplayTag).

## Setting variables in the graph

**SetVariable** comes in two forms — same logic, different visual prominence:

- As a **standalone Action node** in the graph flow (when setting the value is the main step).
- As a **SideEffect Sub-Node** on another node (when it happens as a side effect).

> 📸 **Image placeholder:** `setvariable-as-node.png` — SetVariable node as a standalone box in the graph.
> *Setup:* Graph excerpt. `SayLine "Who are you?"` (output pin) → `SetVariable` node (violet title bar). Details panel on the right: `VariableName = HasAngered`, `Type = Bool`, `Scope = Dialogue`, `Value = true`. Output pin of the SetVariable node leads to another node.

> 📸 **Image placeholder:** `setvariable-as-sideeffect.png` — SetVariable as a SideEffect pill on a SayLine.
> *Setup:* A SayLine selected in the graph. In the node body at the bottom: a SideEffect pill with text `⚙ SetVariable: HasMet = true`. No standalone node in the graph — the pill is visible in the SayLine's body.

## Reading variables in the graph

Reading happens through **Requirements** on choice or Branch nodes:

- **CheckDialogueVariable** — checks a value in the dialogue scope.
- **CheckParticipantVariable** — checks a value in the participant scope.

> 📸 **Image placeholder:** `requirement-check-variable.png` — Branch node with CheckDialogueVariable requirement.
> *Setup:* Branch node in the graph (blue title bar). In the Details panel on the right under "Requirements": one entry `CheckDialogueVariable — ProvocationCount >= 3`. Output pins: True → SayLine "Enough!", False → SayLine (continues as normal). Both pins visible.

## Reading and writing from Blueprint

**Dialogue scope:**

> 📸 **Image placeholder:** `bp-dialogue-variable-read.png` — Blueprint: GetDialogueVariableBool on the active instance.
> *Setup:* BP graph of an event handler. `Get Active Dialogue` (Subsystem) → `Get Dialogue Variable Bool`. Pin `Variable Name = "HasAngered"`. Return value leads to a Branch node. All pins labeled.

```cpp
UMayDialogueInstance* Instance = UMayDialogueSubsystem::Get(this)->GetActiveDialogue();

// Write
Instance->SetDialogueVariableBool("HasAngered", true);
Instance->SetDialogueVariableInt("ProvocationCount", 3);

// Read
bool Angered = Instance->GetDialogueVariableBool("HasAngered");
int32 Count  = Instance->GetDialogueVariableInt("ProvocationCount");
```

**Participant scope:**

```cpp
UMayDialogueParticipant* Part = Actor->FindComponentByClass<UMayDialogueParticipant>();

// Write
Part->SetPersistentBool("HasMet", true);
Part->SetPersistentFloat("Friendship", 12.5f);

// Read with default
bool HasMet      = Part->GetPersistentBool("HasMet", false);
float Friendship = Part->GetPersistentFloat("Friendship", 0.0f);
```

## Reacting to variable changes

Every variable mutation (both scopes) broadcasts `OnVariableChanged`. Your quest system or an analytics logger can hook into this:

```cpp
Instance->OnVariableChanged.AddDynamic(this, &AQuestDirector::HandleVarChanged);

void AQuestDirector::HandleVarChanged(FName VarName, EMayDialogueVariableScope Scope,
                                       EMayDialogueVariableType Type, FString NewValue)
{
    if (VarName == "QuestAccepted" && NewValue == "true")
    {
        QuestSystem->StartQuest("FindTheKey");
    }
}
```

## Known limitations

{% hint style="warning" %}
**SetVariable from the graph currently only writes to the dialogue scope.** For participant scope from the graph: create a custom Blueprint SideEffect.

**Quick guide:**

1. Content Browser → right-click → **Blueprint Class**, parent class: `UMayDialogueSideEffect`.
2. Override the **Execute Side Effect** function (event, not pure).
3. In the event body: call **Get Instigator** or **Get Target** from the `Context` pin to get the desired actor.
4. Call **Get Component by Class → MayDialogueParticipant** on the actor.
5. On the participant component, call **Set Persistent Bool** (or `Set Persistent Int`, `Set Persistent Float`, `Set Persistent String`, `Set Persistent Tag`).

Then add this SideEffect Blueprint as a Sub-Node on your SayLine or Action node.
{% endhint %}

{% hint style="info" %}
**Arithmetic operations** (e.g. `Friendship += 5`) are not directly possible in the graph. Solution: read the current value in a Blueprint SideEffect, calculate, and write it back.
{% endhint %}

## Summary

- **Dialogue scope**: only during the conversation, automatically gone when the dialogue ends.
- **Participant scope**: survives conversations, SaveGame-ready.
- Types: Bool, Int, Float, String, Tag.
- Declare in the Variables panel of the asset editor.
- `OnVariableChanged` as a hook for external systems.

Next: [Emotions & Tags](emotions-tags.md).
