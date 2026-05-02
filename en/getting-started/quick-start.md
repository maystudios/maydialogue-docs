---
description: "Create a dialogue asset, place an NPC in the level, and test: in five minutes."
---

# Quick Start

By the end of this guide you will have a playable NPC dialogue in your level: a guard asks a question, the player picks from two answers, and each leads to a different reaction. No UMG setup, no audio handling, no input fiddling: the plugin takes care of all of that.

Prerequisite: Plugin installed (see [Installation](installation.md)).

---

## Step 1: Create a dialogue asset

1. Open the Content Browser.
2. Right-click in `Content/Dialogues/` (create the folder if it doesn't exist).
3. Choose **May Dialogue → Dialogue Asset**.
4. Name the asset: `DA_Greeting_Simple`.
5. Double-click to open the graph editor.

You will see an empty graph with an **Entry node** (green capsule). The Entry node is always present and is always the starting point of the dialogue.

![Content Browser showing the new Dialogue Asset DA_Greeting_Simple](../../assets/quickstart-01-new-asset.png)

![Empty MayDialogue graph with Entry and Exit node](../../assets/quickstart-02-empty-graph.png)

---

## Step 2: Define speakers

In the **Speakers panel** (sidebar of the asset editor):

**Guard (Speaker 1):**
1. Click **Add Speaker**.
2. Tag: `Dialogue.Speaker.Guard`.
3. DisplayName: `Guard`.
4. NodeColor: Dark red.

**Player (Speaker 2):**
1. Click **Add Speaker**.
2. Tag: `Dialogue.Speaker.Player`.
3. DisplayName: `You`.
4. NodeColor: Grey.

![Speakers panel with two configured speakers: You and Guard](../../assets/quickstart-03-speakers-panel.png)

---

## Step 3: First SayLine

1. Right-click in the graph → choose **Say Line**.
2. Configure the new node in the Details panel (or by double-clicking the node):
   * `SpeakerTag`: `Dialogue.Speaker.Guard`
   * `DialogueText`: `Halt! Who are you?`
3. Connect the Entry output pin to the SayLine input pin.

The node's title bar automatically takes on the color of the selected speaker (dark red).

![Graph with Entry node and first SayLine connected](../../assets/quickstart-04-first-sayline.png)

---

## Step 4: Create a PlayerChoice

1. Right-click in the graph → choose **Player Choice**.
2. Connect the SayLine output pin to the PlayerChoice input pin.
3. In the Details panel: `PromptText` = `You answer:` (optional).
4. Add two elements to the Choices array:
   * Choice 0: Text `A friend of the king.`
   * Choice 1: Text `That's none of your business.`

![PlayerChoice node with two choices in the graph, connected after the SayLine](../../assets/quickstart-06-playerchoice.png)

---

## Step 5: Reaction SayLines

One SayLine for each choice as a reaction:

**SayLine A:**
* `SpeakerTag`: `Dialogue.Speaker.Guard`
* `DialogueText`: `Then pass in peace.`

**SayLine B:**
* `SpeakerTag`: `Dialogue.Speaker.Guard`
* `DialogueText`: `Then get out of here!`

Connect:
* PlayerChoice output pin 0 → SayLine A input pin.
* PlayerChoice output pin 1 → SayLine B input pin.

![Both reaction SayLines connected from PlayerChoice output pins 0 and 1](../../assets/quickstart-07-reactions.png)

---

## Step 6: Exit

1. Right-click in the graph → choose **Exit**.
2. Connect SayLine A output pin → Exit input pin.
3. Connect SayLine B output pin → Exit input pin (same Exit node).

![Complete DA_Greeting_Simple graph: Entry → SayLine → PlayerChoice → two reaction SayLines → Exit](../../assets/quickstart-08-final-graph.png)

---

## Step 7: Compile

Click **Toolbar → Compile**.

If the validator reports errors, fix them in the **Compiler Results** panel:

| Error | Cause | Fix |
| --- | --- | --- |
| Unconnected output pin | A node has no outgoing connection | Connect the output pin to Exit or the next node |
| Empty speaker tag | SayLine without a speaker | Set the SpeakerTag in the Details panel |
| Empty SayLine | No text | Enter text |

---

## Optional: Quick test in the editor: Preview Runner

Before placing any level actors you can already walk through the entire dialogue flow right inside the asset editor: no PIE required.

In the asset editor, find the **Preview** panel (docked below the graph by default). Click the **Play** button to start:

* The preview highlights the active node in the graph as it advances.
* Speaker name and dialogue text appear in the Preview panel.
* When a PlayerChoice node is reached, clickable choice buttons appear: select one to continue.
* Click **Stop** (or let the dialogue reach the Exit node) to end the session.

![Preview panel running the DA_Greeting_Simple dialogue in the editor with the active node highlighted in the graph](../../assets/quickstart-preview-runner.png)

This is the fastest way to verify flow and text before touching the level. Requirements and side effects that depend on GAS or persistent variables are skipped in the preview, but the branching structure is fully testable.

---

## Step 8: Add the Participant component to the NPC and player pawn

**Guard actor:**

1. Open your level.
2. Place any actor as a guard placeholder in the level (e.g. a Character Blueprint or a StaticMesh actor).
3. In the Details panel: **Add Component → MayDialogue Participant**.
4. Configure the component:
   * `ParticipantTag`: `Dialogue.Speaker.Guard`
   * `DisplayName`: `Guard`
   * `DefaultDialogue`: `DA_Greeting_Simple`

**Player pawn:**

1. Open your player pawn Blueprint (e.g. `BP_ThirdPersonCharacter`) in the Blueprint editor.
2. In the **Components** panel at the top left: **Add Component → MayDialogue Participant**.
3. Configure the component:
   * `ParticipantTag`: `Dialogue.Speaker.Player`

The plugin needs this component to know who the instigator of the dialogue is: even if the player has no SayLines of their own in this Quick Start.

![Details panel of the guard actor showing the MayDialogueParticipant component with ParticipantTag, DisplayName and DefaultDialogue filled in](../../assets/quickstart-09-participant-component.png)

---

## Step 9: Trigger the dialogue

{% hint style="success" %}
**Recommended approach for Blueprint users: Option A**
{% endhint %}

**Option A: directly via the participant (recommended):**

In the Blueprint graph of your trigger actor or player logic:

1. Get a reference to the guard actor.
2. Call **Get Component by Class: MayDialogueParticipant**.
3. On the participant: call **Start Default Dialogue**.
4. `Other` parameter: reference to the player's participant component.

![Blueprint graph of a trigger actor: OnComponentBeginOverlap → Get MayDialogueParticipant → Start Default Dialogue with player participant as Other](../../assets/quickstart-10-blueprint-trigger.png)

**Option B: via the library function:**

![Blueprint graph calling StartDialogue on the MayDialogue Subsystem with DA_Greeting_Simple, Get Player Pawn as Instigator, and Self as Target](../../assets/quickstart-10b-library-trigger.png)

```
MayDialogueLibrary → StartDialogue
  WorldContext = Self
  Asset        = DA_Greeting_Simple
  Instigator   = PlayerPawn
  Target       = GuardActor
```

<details>
<summary>For C++ users</summary>

```cpp
UMayDialogueParticipant* GuardParticipant =
    Guard->FindComponentByClass<UMayDialogueParticipant>();
if (GuardParticipant)
{
    GuardParticipant->StartDefaultDialogue(Player);
}
```

</details>

---

## Step 10: Test

**Start PIE**, walk up to the guard, and trigger the trigger.

What you see:
* A widget appears in the viewport (Slate debug widget, as long as no UMG widget is set).
* The guard speaks: *"Halt! Who are you?"*.
* Two choice buttons appear.
* Clicking a choice leads to the matching reaction, and the dialogue ends.

![In-game PIE screenshot showing the Slate debug widget with speaker name, dialogue text and two choice buttons](../../assets/quickstart-11-ingame.png)

{% hint style="success" %}
You now have a playable dialogue with no UMG setup, no audio configuration, and no input handling code.
{% endhint %}

---

## Troubleshooting

<details>
<summary>No widget appears when the dialogue starts</summary>

Check in **Edit → Project Settings → Plugins → MayDialogue** that `bUseSlateDialogueWidget` is enabled. By default the Slate debug widget appears even without a UMG widget set.
</details>

<details>
<summary>Dialogue doesn't start at all</summary>

Check:
* Does the asset have an Entry node and has it been compiled (Toolbar → Compile)?
* Does the guard actor have a `MayDialogueParticipant` component with the correct tag?
* Does the player pawn also have a participant component?
* The Output Log shows warnings when something is missing: search for `MayDialogue` there.
</details>

<details>
<summary>The question appears but no choice buttons</summary>

This happens when the PlayerChoice node has no connected output pins, or all choices are hidden due to requirements. Open the asset and check the choices list on the PlayerChoice node.
</details>

---

## What's next?

* Learn about variables, branching, and GAS requirements → [Walkthrough](first-dialogue.md)
* Use a custom UMG widget instead of the Slate debug widget → [UI Architecture](../ui/umg-architecture.md)
* Add audio → [Audio System](../audio/README.md)
* Tie choices to GAS attributes → [GAS Integration](../gas/README.md)
