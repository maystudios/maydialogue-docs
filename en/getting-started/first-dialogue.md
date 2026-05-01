---
description: Variables, choice requirements, and SideEffect actions — a complete dialogue.
---

# Walkthrough — A Complete Dialogue

The [Quick Start](quick-start.md) showed you how to build a playable dialogue in five minutes. This walkthrough goes one step further. You will build a realistic NPC conversation that uses variables, ties choices to GAS tags and attributes, and triggers SideEffect actions.

{% hint style="warning" %}
**Prerequisites for steps 6–7 (GAS attribute and ApplyEffect):**

- The **Gameplay Ability System** (GAS) must be enabled in your project.
- You need a custom `UAttributeSet` with the attribute `Reputation.Guards` on the player's ASC.
- You need a GameplayEffect `GE_GuardTrust` that increases this attribute.

**No GAS in your project?** Skip step 6 (requirement on Choice 1) and step 7 (ApplyEffect) and replace them with simple variable checks or SetVariable SideEffects from the dialogue scope. A non-GAS variant is described in the [Optional: Variable Variant Without GAS](#optional-variable-variant-without-gas) section at the end of this walkthrough.
{% endhint %}

## The scenario

You meet a guard in front of a fortress. Three things determine what happens:

1. **Have you met him before?** — a participant variable `HasMet` (Bool).
2. **Have you heard the password?** — a GameplayTag `Story.Secret.HeardPassword` on the player.
3. **What is your reputation?** — a GAS attribute `Reputation.Guards` on the player.

---

## Step 1 — Create a new dialogue asset

1. Content Browser → `Content/Dialogues/` → Right-click → **Miscellaneous → MayDialogue Asset**.
2. Name it: `DA_Gate_Guardian`.
3. Double-click to open the graph editor.

### Create speakers

In the **Speakers panel**:

| Tag | DisplayName | Color |
| --- | --- | --- |
| `Dialogue.Speaker.Guard` | Guard | Dark red |
| `Dialogue.Speaker.Player` | You | Grey |

### Create a participant variable

In the **Variables panel** (sidebar):

1. Click **Add Variable**.
2. Name: `HasMet`, Type: `Bool`, Scope: `Participant`, Default: `false`.

Participant scope means: the variable belongs to the guard actor and persists between dialogue starts (as long as the actor lives in the level).

> 📸 **Image placeholder:** `walkthrough-01-variables-panel.png` — Variables panel with the HasMet variable.
> *Setup:* Variables panel in the asset editor, one entry visible: `HasMet`, type `Bool`, scope `Participant`, default `false`. Red arrow on the scope dropdown.

---

## Step 2 — Greeting with Branch

The entry point asks: has the player met the guard before?

1. Right-click in the graph → place a **Branch** node.
2. Connect Entry output pin → Branch input pin.
3. Add a Requirement Sub-Node to the Branch node:
   * Sub-node type: **CheckParticipantVariable**
   * `VariableName`: `HasMet`
   * `ExpectedValue`: `true`
   * `CheckTarget`: `Target` (the guard is the target of the dialogue)

**True output** → **SayLine** "So you're back again. What do you want?"
**False output** → **SayLine** "Halt! Who are you?"

> 📸 **Image placeholder:** `walkthrough-02-branch-node.png` — Branch node with CheckParticipantVariable Sub-Node in the graph.
> *Setup:* Graph shows: `Entry` → `Branch` (diamond shape with True, False, and Fallback pins). In the Branch node body the Sub-Node "CheckParticipantVariable: HasMet == true" is visible as a pill. From the True pin an arrow goes to SayLine "So you're back again." (dark red), from the False pin to SayLine "Halt! Who are you?" (dark red). Fallback pin not connected (shows empty pin).

> 📸 **Image placeholder:** `walkthrough-03-branch-subnodes.png` — Details panel of the CheckParticipantVariable requirement.
> *Setup:* Branch node selected, in the Details panel the embedded requirement is visible: `VariableName = HasMet`, `ExpectedValue = true`, `CheckTarget = Target`. Red arrow on `CheckTarget`.

---

## Step 3 — Set HasMet via SideEffect

Each greeting SayLine (True and False path) should set `HasMet = true` when entered — regardless of which variant is played.

**On both greeting SayLines**, do each of the following:

1. Select the SayLine.
2. In the Sub-Nodes area: **Add SideEffect → SetVariable**.
3. `VariableName`: `HasMet`, `NewValue`: `true`, `Scope`: `Participant`, `Target`: guard participant.

The SideEffect appears as a pill in the SayLine's node body — visible but unobtrusive.

> 📸 **Image placeholder:** `walkthrough-04-sideeffect-pill.png` — SayLine "Halt! Who are you?" with a SideEffect pill in the node body.
> *Setup:* SayLine node "Halt! Who are you?" in the graph. In the lower area of the node body a pill is visible: small icon (pen or variable symbol) + text "SetVariable: HasMet = true". Red arrow pointing at the pill.

---

## Step 4 — PlayerChoice with three options

Both greeting SayLines feed into the same **PlayerChoice** node:

1. Connect True SayLine output → PlayerChoice input.
2. Connect False SayLine output → PlayerChoice input.
3. Create three elements in the Choices array:
   * Choice 0: `I know the password: Elenderion.`
   * Choice 1: `I'm a friend of the king.`
   * Choice 2: `That's none of your business.`

> 📸 **Image placeholder:** `walkthrough-05-playerchoice-overview.png` — PlayerChoice node with three choices and two incoming arrows.
> *Setup:* Graph shows SayLine "Halt!" (upper left) and SayLine "So you're back again." (lower left), both with arrows to the PlayerChoice node (center-right). In the PlayerChoice body three choice entries as pills: "I know the password…", "I'm a friend…", "That's none of your business." Three output pins on the right (0, 1, 2).

---

## Step 5 — Requirement on Choice 0 (password)

Choice 0 should only appear if the player has the tag `Story.Secret.HeardPassword`.

1. Select Choice 0.
2. **Add Requirement → HasTag**.
3. `RequiredTag`: `Story.Secret.HeardPassword`.
4. `CheckOnInstigator`: `true` (the player is the instigator).
5. `FailureResult`: `FailedAndHidden` — the choice disappears entirely if the condition is not met.

> 📸 **Image placeholder:** `walkthrough-06-requirement-hastag.png` — Details panel of the HasTag requirement on Choice 0.
> *Setup:* Choice 0 of the PlayerChoice node selected. Details panel shows: `RequiredTag = Story.Secret.HeardPassword`, `CheckOnInstigator = true`, `FailureResult = FailedAndHidden`. Red arrow on `FailureResult`.

This produces the classic RPG pattern: you only see the password option if you have actually heard the password.

---

## Step 6 — Requirement on Choice 1 (reputation)

Choice 1 should be visible but not selectable if reputation is too low.

1. Select Choice 1.
2. **Add Requirement → CheckAttribute**.
3. `Attribute`: `Reputation.Guards`.
4. `ComparisonOp`: `>=`.
5. `ComparisonValue`: `50`.
6. `CheckOnInstigator`: `true`.
7. `FailureResult`: `FailedButVisible` — the choice appears greyed out but cannot be selected.
8. `UnavailableReason`: `Your reputation with the guards is too low.` (shown as a tooltip).

> 📸 **Image placeholder:** `walkthrough-07-requirement-attribute.png` — Details panel of the CheckAttribute requirement on Choice 1.
> *Setup:* Choice 1 of the PlayerChoice node selected. Details panel shows: `Attribute = Reputation.Guards`, `ComparisonOp = >=`, `ComparisonValue = 50`, `CheckOnInstigator = true`, `FailureResult = FailedButVisible`, `UnavailableReason = "Your reputation with the guards is too low."`. Red arrow on `FailedButVisible`.

> 📸 **Image placeholder:** `walkthrough-08-ingame-choices.png` — In-game screenshot of the PlayerChoice widget with a greyed-out choice.
> *Setup:* PIE running. Widget shows three choices: Choice 0 is missing (because HasHeardPassword is not set), Choice 1 is visible but greyed out with tooltip "Your reputation with the guards is too low.", Choice 2 is normally clickable "That's none of your business." Red arrow on the greyed-out Choice 1.

---

## Step 7 — Consequences: Choice 0 with ApplyEffect

**Path for Choice 0 (password):**

After output pin 0 of the PlayerChoice, place:

1. **Action node: Apply Effect**
   * `EffectClass`: `GE_GuardTrust` (your project's GameplayEffect that increases reputation)
   * `EffectLevel`: `1.0`
   * `ApplyToInstigator`: `true`
2. **SayLine**: `Then pass in peace.` (Speaker: Guard)
3. **Exit** (Status: Completed)

Connections: `PlayerChoice OutputPin[0]` → `ApplyEffect` → `SayLine "Pass in peace"` → `Exit`.

> 📸 **Image placeholder:** `walkthrough-09-choice0-path.png` — Path from Choice 0 with ApplyEffect node in the graph.
> *Setup:* To the right of PlayerChoice, top: arrow from output pin 0 → `ApplyEffect` node (body shows "GE_GuardTrust, Level 1.0, Instigator") → `SayLine "Then pass in peace."` (dark red) → `Exit` (red capsule, Status: Completed). All connection arrows visible.

---

## Step 8 — Consequences: Choice 1 and Choice 2

**Path for Choice 1 (king):**

`PlayerChoice OutputPin[1]` → `SayLine "Then pass, friend."` → `Exit (Completed)`.

**Path for Choice 2 (rude):**

Place:
1. **Action node: Add Tag** — `Tag`: `Story.Guard.Hostile`, `AddToInstigator`: `false` (tag is added to the guard)
2. **Action node: Camera Shake** — your project's `UCameraShakeBase`, `Scale: 1.5`
3. **SayLine** `Then get out of here!` (Speaker: Guard)
4. **Exit** (Status: Failed)

Connections: `PlayerChoice OutputPin[2]` → `AddTag` → `CameraShake` → `SayLine "Get out"` → `Exit (Failed)`.

> 📸 **Image placeholder:** `walkthrough-10-choice2-path.png` — Path from Choice 2 with AddTag and CameraShake nodes.
> *Setup:* To the right of PlayerChoice, bottom: arrow from output pin 2 → `AddTag` node (body: "Story.Guard.Hostile → Target") → `CameraShake` node (body: "Scale 1.5") → `SayLine "Then get out!"` (dark red) → `Exit (Failed)` (red capsule, Status: Failed). All arrows labeled.

---

## Step 9 — RandomLine for repeated visits

The SayLine "So you're back again." becomes repetitive after the third visit. Replace it with a **Random Line** node:

1. Delete the SayLine "So you're back again."
2. Right-click → place **Random Line**.
3. Enter three lines in the Details panel:
   * `So you're back again. What do you want?`
   * `Back? Hope you have a good reason.`
   * `Again. What is it this time?`
4. `bRememberSelection`: `true` — prevents two identical lines playing in a row.
5. Connect True path of Branch → RandomLine input.
6. Connect RandomLine output → PlayerChoice input (as before).

> 📸 **Image placeholder:** `walkthrough-11-random-line.png` — RandomLine node in the graph with a dice icon and three lines in the body.
> *Setup:* RandomLine node (dice icon in the title bar). In the node body three lines visible as pills: "So you're back again…", "Back?…", "Again…". Details panel shows `bRememberSelection = true`. Arrow incoming from Branch True pin, arrow outgoing to PlayerChoice.

---

## Step 10 — Compile and test

1. Click **Toolbar → Compile**. No errors in the Compiler Results panel?
2. Start PIE.

**Test matrix:**

| State | Expected behavior |
| --- | --- |
| First visit, no password, reputation < 50 | Greeting "Halt!"; Choice 0 missing; Choice 1 greyed; only Choice 2 selectable |
| First visit, password heard, reputation >= 50 | All three choices visible and selectable |
| Second visit | RandomLine greeting instead of "Halt!" |
| Choice 0 selected | ApplyEffect increases reputation; guard bids you farewell kindly |
| Choice 2 selected | `Story.Guard.Hostile` tag set; CameraShake; guard throws you out |

> 📸 **Image placeholder:** `walkthrough-12-full-graph-overview.png` — Overview of the complete graph DA_Gate_Guardian.
> *Setup:* Full graph from a bird's-eye view (zoomed out). Visible left to right: `Entry` → `Branch` → True path (RandomLine → PlayerChoice) and False path (SayLine "Halt!" → PlayerChoice). From PlayerChoice three outputs to the right: top Choice-0-path (ApplyEffect → SayLine → Exit Completed), middle Choice-1-path (SayLine → Exit Completed), bottom Choice-2-path (AddTag → CameraShake → SayLine → Exit Failed). SideEffect pills on the greeting nodes visible.

---

## What you built

| Feature | Used in |
| --- | --- |
| Participant variable | `HasMet` — remembers the first visit |
| Branch node + requirement | Greeting branch based on `HasMet` |
| SideEffect → SetVariable | Set `HasMet` to `true` on first entry |
| PlayerChoice with requirements | GAS tag- and attribute-driven options |
| Action node: ApplyEffect | Increase reputation after password |
| Action node: AddTag | Mark guard as `Story.Guard.Hostile` |
| Action node: CameraShake | Dramatic moment when being thrown out |
| RandomLine | Varied greeting on repeated visits |

---

## Next steps

* [Core Concepts](../concepts/README.md) — understand the mental model
* [Node Reference](../nodes/README.md) — all nodes in detail
* [Recipes](../recipes/README.md) — more examples for common patterns

{% hint style="info" %}
**Building custom requirement types:** Create a Blueprint class with parent `UMayDialogueRequirement`. In the `IsRequirementSatisfied` function, return `Passed`, `FailedButVisible`, or `FailedAndHidden`. The new type immediately appears in the Sub-Node palette. See [Custom Requirements](../extension/custom-requirements.md).
{% endhint %}

---

## Optional: Variable Variant Without GAS

If GAS is not active in your project, you can replace steps 6 and 7 with variable-based alternatives:

**Instead of step 6 (CheckAttribute requirement on Choice 1):**

Create a dialogue variable `Reputation` (Type: `Int`, Default: `0`) in the Variables panel. Use **CheckDialogueVariable** as the requirement with `VariableName = Reputation`, `ComparisonOp = >=`, `ComparisonValue = 50`.

**Instead of step 7 (ApplyEffect):**

Replace the Apply Effect node with a **Set Variable** node: `VariableName = Reputation`, `NewValue = Reputation + 50`. Since arithmetic operations are not available directly in the graph, you can create a small Blueprint SideEffect for this (see [Variables & Scopes](../concepts/variables-scopes.md)).

The test matrix and the rest of the walkthrough remain identical.
