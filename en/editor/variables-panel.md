---
description: Creating variables, choosing types and scope, setting default values, and using them in the graph.
---

# Variables Panel

The Variables panel is where you declare **all variables of a dialogue asset**. Variables let you store state within a dialogue and make branching dependent on it.

> 📸 **Image placeholder:** `variables-panel-overview.png` — Variables panel with four entries of different types and scopes.
> *Setup:* Variables tab active. Four rows: `HasAskedName` (Bool, Dialogue scope, Default: false), `AngerLevel` (Int, Dialogue scope, Default: 0), `Friendship` (Float, Participant scope, Default: 0.5), `SelectedTone` (Tag, Dialogue scope, empty). All columns visible: Name, Type, Scope, Default, Description.

## Creating a variable

1. Open the Variables tab.
2. Click **Add Variable**.
3. Type a **Name** (unique within the chosen scope).
4. Choose a **Type** from the dropdown.
5. Set the **Scope**: Dialogue or Participant.
6. Set the **Default value**.
7. Optionally: enter a **Description** for yourself or other designers.

> 📸 **Image placeholder:** `variables-panel-add-variable.png` — "Add Variable" button clicked, new row with cursor in the Name field.
> *Setup:* Variables panel with two existing variables. Third row newly created, name field active, type dropdown closed (Bool as default). Red arrow on "Add Variable" button.

## Available types

| Type | For | Default example |
| --- | --- | --- |
| **Bool** | Yes/no flags | `false` |
| **Int** | Counters, levels | `0` |
| **Float** | Percentage values, metrics | `0.0` |
| **String** | Free text (rarely used in graph) | `""` |
| **Tag** | GameplayTag selection | *(empty)* |

## Choosing a scope

**Dialogue scope** — the variable lives only for the duration of this dialogue run.

**Participant scope** — the variable is bound to a participant and persists after the dialogue ends (as long as the participant is in memory).

### Decision guide

```
Question: Should the NPC still remember this after the dialogue?
├─ No  → Dialogue scope
└─ Yes → Participant scope
```

**Typical dialogue scope variables:**
- `AngerLevel: Int` — counts provocations during the conversation
- `HasAskedName: Bool` — prevents asking the same question twice
- `SelectedTone: Tag` — remembers the chosen tone

**Typical participant scope variables:**
- `HasMet: Bool` — first contact stored
- `Friendship: Float` — relationship metric
- `KnownSecrets: Tag` — which secrets the NPC knows

> 📸 **Image placeholder:** `variables-panel-scope-dropdown.png` — Scope dropdown of a variable open, "Dialogue" and "Participant" as options.
> *Setup:* Variable `HasMet` selected. Scope field expanded as dropdown, two options visible. Red arrow on the dropdown.

## Using variables in the graph

### Reading (Requirement Sub-Nodes)

You read variable values through **Requirement Sub-Nodes** on Branch or Choice nodes:

- `CheckDialogueVariable` — checks a dialogue scope variable
- `CheckParticipantVariable` — checks a participant scope variable

Choose the Sub-Node type, enter the variable name, set the comparison condition.

### Writing (SetVariable)

Two ways to set a value:

**As an Action node in the flow:**
Right-click in the graph → Actions → Set Variable. The node appears as a full box in the flow.

**As a SideEffect Sub-Node:**
Right-click on a SayLine or Choice node → Add SideEffect → Set Variable. More compact representation as a pill on the parent node — it happens in the background when the parent node is executed.

```text
Example graph: Anger tracking
[Entry] → [SayLine: "You fool!"]
          └─ SideEffect: SetVariable(AngerLevel = AngerLevel + 1)
        → [Branch: CheckDialogueVariable(AngerLevel >= 3)]
          ├─ True  → [SayLine: "Enough! Out!"] → [Exit: Failed]
          └─ False → [SayLine: "Once more..."] → [back]
```

> 📸 **Image placeholder:** `variables-setvariable-node.png` — SetVariable Action node in the graph, Details panel shows variable name and value.
> *Setup:* SetVariable node selected. Details show: VariableName = "AngerLevel", VariableType = Int, NewValue = 1. In the graph beside it: connection from SayLine to this SetVariable node to a Branch node.

## Validator rule: Type Mismatch

If two SetVariable nodes describe the same name with different types, the compiler reports a **Variable Type Mismatch** error.

```
Variable "AngerLevel" declared as Int.
SetVariable node A: writes 1      (Int)    → OK
SetVariable node B: writes "High" (String) → ERROR
```

Always declare the variable in the Variables panel with the correct type — this prevents this error.

## Variables in Preview and Debugger

In the **Preview Runner** you see all dialogue scope variables in real time and can manually set their values to test branches — without simulating real player actions.

In the **PIE Debugger**, the Debugger Watch tab shows all active variables of the running instance: dialogue scope values directly, participant scope values from the bound actors.

See [Preview Runner](preview-runner.md) and [Debugger](debugger.md) for details.

> 📸 **Image placeholder:** `variables-preview-watch.png` — Preview Runner state area with two variables shown live.
> *Setup:* Preview running, dialogue has reached AngerLevel=1. At the bottom of the Preview panel: variable list shows `HasAskedName: false`, `AngerLevel: 1`. Red arrow pointing at the variable list in the Preview state area.

Next: [Outline →](outline.md)
