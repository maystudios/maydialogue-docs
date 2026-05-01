---
description: Create a dialogue asset, place an NPC in the level, and test — in five minutes.
---

# Quick Start

By the end of this guide you will have a playable NPC dialogue in your level: a guard asks a question, the player picks from two answers, and each leads to a different reaction. No UMG setup, no audio handling, no input fiddling — the plugin takes care of all of that.

Prerequisite: Plugin installed (see [Installation](installation.md)).

---

## Step 1 — Create a dialogue asset

1. Open the Content Browser.
2. Right-click in `Content/Dialogues/` (create the folder if it doesn't exist).
3. Choose **May Dialogue → Dialogue Asset**.
4. Name the asset: `DA_Greeting_Simple`.
5. Double-click to open the graph editor.

You will see an empty graph with an **Entry node** (green capsule). The Entry node is always present and is always the starting point of the dialogue.

> 📸 **Image placeholder:** `quickstart-01-new-asset.png` — Content Browser showing the new asset `DA_Greeting_Simple`.
> *Setup:* Content Browser, folder `Content/Dialogues/`, the new asset with the MayDialogue icon visible. Red arrow pointing at the asset.

> 📸 **Image placeholder:** `quickstart-02-empty-graph.png` — Empty MayDialogue graph with Entry node.
> *Setup:* Graph editor open with asset `DA_Greeting_Simple`. Visible: only the Entry node (green compact capsule) in the center of the graph, otherwise empty canvas. Speakers panel on the right is empty, Outline panel on the left is empty.

---

## Step 2 — Define speakers

In the **Speakers panel** (sidebar of the asset editor):

**Speaker 1 — Guard:**
1. Click **Add Speaker**.
2. Tag: `Dialogue.Speaker.Guard`.
3. DisplayName: `Guard`.
4. NodeColor: Dark red.

**Speaker 2 — Player:**
1. Click **Add Speaker**.
2. Tag: `Dialogue.Speaker.Player`.
3. DisplayName: `You`.
4. NodeColor: Grey.

> 📸 **Image placeholder:** `quickstart-03-speakers-panel.png` — Speakers panel with two configured speakers.
> *Setup:* The Speakers panel on the right in the asset editor. Two entries visible: "Guard" with a dark red color chip and tag `Dialogue.Speaker.Guard`; "You" with a grey color chip and tag `Dialogue.Speaker.Player`. Both entries fully expanded, all fields visible.

---

## Step 3 — First SayLine

1. Right-click in the graph → choose **Say Line**.
2. Configure the new node in the Details panel (or by double-clicking the node):
   * `SpeakerTag`: `Dialogue.Speaker.Guard`
   * `DialogueText`: `Halt! Who are you?`
3. Connect the Entry output pin to the SayLine input pin.

The node's title bar automatically takes on the color of the selected speaker (dark red).

> 📸 **Image placeholder:** `quickstart-04-first-sayline.png` — Graph with Entry node and first SayLine connected.
> *Setup:* Graph shows left to right: `Entry` node (green capsule), arrow to `SayLine` node (dark red title bar, text "Halt! Who are you?" visible in the node body, speaker: Guard). Connection arrow between Entry output pin and SayLine input pin visible.

> 📸 **Image placeholder:** `quickstart-05-sayline-properties.png` — Details panel of the first SayLine.
> *Setup:* SayLine node selected. Details panel on the right shows: `SpeakerTag = Dialogue.Speaker.Guard`, `DialogueText = "Halt! Who are you?"`, `AdvanceModeOverride = (none)`. Red arrows pointing at SpeakerTag and DialogueText.

---

## Step 4 — Create a PlayerChoice

1. Right-click in the graph → choose **Player Choice**.
2. Connect the SayLine output pin to the PlayerChoice input pin.
3. In the Details panel: `PromptText` = `You answer:` (optional).
4. Add two elements to the Choices array:
   * Choice 0: Text `A friend of the king.`
   * Choice 1: Text `That's none of your business.`

> 📸 **Image placeholder:** `quickstart-06-playerchoice.png` — PlayerChoice node with two choices in the graph.
> *Setup:* Graph shows the chain `Entry → SayLine → PlayerChoice`. The PlayerChoice node is wider than the SayLine, and in the body two Choice Sub-Nodes are visible as pills: "A friend of the king." and "That's none of your business." Two output pins on the right of the node (Pin 0 and Pin 1).

---

## Step 5 — Reaction SayLines

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

> 📸 **Image placeholder:** `quickstart-07-reactions.png` — Both reaction SayLines in the graph with connections from the PlayerChoice.
> *Setup:* Graph shows `PlayerChoice` with two outgoing arrows: output pin 0 → SayLine A "Then pass in peace." (dark red); output pin 1 → SayLine B "Then get out of here!" (dark red). Both SayLines to the right of the PlayerChoice, connection lines clearly visible labeled "0" and "1".

---

## Step 6 — Exit

1. Right-click in the graph → choose **Exit**.
2. Connect SayLine A output pin → Exit input pin.
3. Connect SayLine B output pin → Exit input pin (same Exit node).

Your graph now looks like this:

```
[Entry] → [SayLine: "Halt!"] → [PlayerChoice]
                                 ├─ Pin 0 → [SayLine: "Pass in peace"] ─┐
                                 └─ Pin 1 → [SayLine: "Get out"]        ─┤
                                                                        [Exit]
```

> 📸 **Image placeholder:** `quickstart-08-final-graph.png` — Finished graph with all nodes and connections.
> *Setup:* Overview of the complete graph `DA_Greeting_Simple`. Left to right: `Entry` (green capsule) → `SayLine "Halt! Who are you?"` (dark red) → `PlayerChoice` (wide, 2 output pins) → top `SayLine "Then pass in peace"` (dark red) and bottom `SayLine "Then get out!"` (dark red) → both arrows meet at the shared `Exit` (red capsule). Horizontal layout, all connection arrows clearly visible.

---

## Step 7 — Compile

Click **Toolbar → Compile**.

If the validator reports errors, fix them in the **Compiler Results** panel:

| Error | Cause | Fix |
| --- | --- | --- |
| Unconnected output pin | A node has no outgoing connection | Connect the output pin to Exit or the next node |
| Empty speaker tag | SayLine without a speaker | Set the SpeakerTag in the Details panel |
| Empty SayLine | No text | Enter text |

---

## Step 8 — Add the Participant component to the NPC and player pawn

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
   * `ParticipantTag`: `Player.Character`

The plugin needs this component to know who the instigator of the dialogue is — even if the player has no SayLines of their own in this Quick Start.

> 📸 **Image placeholder:** `quickstart-09-participant-component.png` — Details panel of the guard actor with the MayDialogueParticipant component.
> *Setup:* Details panel of a level actor. In the component list, `MayDialogueParticipant` is visible (selected). Below it the properties: `ParticipantTag = Dialogue.Speaker.Guard`, `DisplayName = Guard`, `DefaultDialogue = DA_Greeting_Simple`. Red arrow pointing at `DefaultDialogue`.

---

## Step 9 — Trigger the dialogue

{% hint style="success" %}
**Recommended approach for Blueprint users: Option A**
{% endhint %}

**Option A — directly via the participant (recommended):**

In the Blueprint graph of your trigger actor or player logic:

1. Get a reference to the guard actor.
2. Call **Get Component by Class: MayDialogueParticipant**.
3. On the participant: call **Start Default Dialogue**.
4. `Other` parameter: reference to the player's participant component.

> 📸 **Image placeholder:** `quickstart-10-blueprint-trigger.png` — Blueprint graph of a trigger actor with the StartDialogue call.
> *Setup:* BP graph of a box trigger actor. Event `OnComponentBeginOverlap` → `Get Component by Class (MayDialogueParticipant)` on the guard actor → `Start Default Dialogue` with `Other` = player participant reference. All pins labeled, execution arrows visible.

**Option B — via the library function:**

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

## Step 10 — Test

**Start PIE**, walk up to the guard, and trigger the trigger.

What you see:
* A widget appears in the viewport (Slate debug widget, as long as no UMG widget is set).
* The guard speaks: *"Halt! Who are you?"*.
* Two choice buttons appear.
* Clicking a choice leads to the matching reaction, and the dialogue ends.

> 📸 **Image placeholder:** `quickstart-11-ingame.png` — In-game screenshot of the running dialogue in PIE.
> *Setup:* PIE active. In the viewport the Slate debug widget is visible: speaker name "Guard" at the top, text "Halt! Who are you?" below it, two choice buttons "A friend of the king." and "That's none of your business." at the bottom. The guard actor visible in the background.

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
* The Output Log shows warnings when something is missing — search for `MayDialogue` there.
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
