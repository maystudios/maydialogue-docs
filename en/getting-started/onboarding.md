---
description: "The fastest possible first dialogue — and the editor comfort features that get you there."
---

# Onboarding & Editor Comfort

You just built a dialogue the long way in the [Quick Start](quick-start.md). This page shows the **fast** way: the scaffolding, templates, and editor hints that take you from "new project" to "talking NPC in PIE" in about five minutes — most of it without writing or wiring anything.

If you only read one onboarding page, read this one. It also clears up the single most common first-time confusion (the UMG widget footgun) before it ever bites you.

{% hint style="success" %}
**Where this fits:** do the [Quick Start](quick-start.md) once to understand the moving parts, then use the shortcuts here every time after. When you want the full feature tour — variables, GAS, requirements — go to the [Walkthrough](first-dialogue.md).
{% endhint %}

---

## Your first 5 minutes

A guided path that skips the manual graph wiring:

1. **Create a dialogue asset** — it opens pre-wired with a starter scaffold, not an empty graph.
2. **Drag a "Dialogue NPC" into your level** from the Place Actors menu.
3. **Assign your dialogue** to the NPC's `DefaultDialogue` slot.
4. **Press Play.** Walk up, talk.

Each step is detailed below.

> 📸 **Image placeholder:** `onboarding-five-minute-flow.png` — The four-step onboarding flow as a strip.
> *Setup:* Simple horizontal graphic with four labeled boxes left to right: "New Asset (scaffolded)" → "Drag Dialogue NPC" → "Assign DefaultDialogue" → "Press Play". Arrows between them. Caption: "From new project to talking NPC in four steps."

---

## Step 1: New assets open with a starter scaffold

When you create a new **May Dialogue → Dialogue Asset**, it no longer opens as a blank canvas with a lone Entry node. It opens with a working **Entry → SayLine → Exit** scaffold already placed and connected.

That means a brand-new asset is **already valid and already runnable**: it compiles, it has a starting line you can edit in place, and it ends cleanly. You replace the placeholder text on the SayLine, hit Compile, and you have a one-line dialogue — no node-by-node wiring required.

> 📸 **Image placeholder:** `onboarding-starter-scaffold.png` — A freshly created asset showing the pre-wired scaffold.
> *Setup:* Asset editor open on a just-created `DA_New_Dialogue`. The graph shows Entry (green) → SayLine (with the seeded placeholder text "Hello! Edit me in the Details panel.") → Exit (red), all connected with visible wires. Caption: "New assets are runnable from the first second."

{% hint style="info" %}
The scaffold is just a normal graph. Delete, rewire, or extend it like any other — there is nothing special about the starter nodes once the asset exists.
{% endhint %}

---

## Step 2: The "Dialogue NPC" template in Place Actors

Open the **Place Actors** panel (Window → Place Actors) and search for **Dialogue NPC** — or open the dedicated **MayDialogue** category tab, where the template is registered. Drag it into your level.

This is a ready-made actor that already carries a `MayDialogueParticipant` component — so you do not have to add the component, set it up, or know which fields matter. It drops in as a talkable NPC out of the box.

> 📸 **Image placeholder:** `onboarding-place-actors-dialogue-npc.png` — The Place Actors panel with the Dialogue NPC template.
> *Setup:* Editor with the Place Actors panel open, "Dialogue" typed into its search field, the **Dialogue NPC** entry highlighted in the results list. A red arrow points to the entry. Caption: "Drag a talkable NPC straight into the level."

---

## Step 3: Assign `DefaultDialogue`

Select the Dialogue NPC you just placed. In the **Details** panel, find its `MayDialogueParticipant` component and set **`DefaultDialogue`** to the asset from Step 1.

That is the only field you must set. The participant tag and display name come pre-filled on the template; you can override them later when you have more than one NPC.

> 📸 **Image placeholder:** `onboarding-assign-default-dialogue.png` — The participant component with DefaultDialogue assigned.
> *Setup:* Details panel of the placed Dialogue NPC, the MayDialogueParticipant component expanded, `DefaultDialogue` set to `DA_New_Dialogue`. A red arrow points to the DefaultDialogue field. Caption: "One field connects the NPC to its dialogue."

---

## Step 4: Press Play

Start PIE, walk up to the NPC, and trigger it. The Slate debug widget appears, the NPC speaks your line, and you have a working dialogue with zero Blueprint and zero code.

When you want to trigger the dialogue from your own logic (an interaction prompt, a trigger volume) instead of relying on the template's default, follow the trigger options in [Quick Start → Step 9](quick-start.md#step-9-trigger-the-dialogue).

---

## The Quick Start assistant

MayDialogue ships a lightweight **Quick Start assistant**, not a full interactive step-by-step tutorial. It is two small touches that point you at this documentation:

- **A first-open orientation notification.** The first dialogue editor you open in a session — and only when the asset still looks like the freshly created starter scaffold — shows a one-time toast: *"New here? The graph already contains a starter scaffold — double-click the Say Line to edit its text."* It carries an **Open Quick Start** link that opens this documentation in your browser. It is fire-and-forget (it fades on its own) and appears at most once per editor session.
- **A Help-menu entry.** The asset editor's **Help** menu has an **Open Quick Start Guide** item that opens this documentation any time.

That is the whole assistant — an orientation nudge plus a deep link, not an overlay that walks you through the panels. The hands-on, step-by-step walkthrough is the written [Quick Start](quick-start.md), which the assistant points you to.

{% hint style="info" %}
There is deliberately **no** interactive in-editor tutorial overlay in 1.0 — a full guided walkthrough was descoped as too heavy. The starter scaffold plus this documentation are the onboarding path.
{% endhint %}

> 📸 **Image placeholder:** `onboarding-quickstart-notification.png` — The first-open orientation notification.
> *Setup:* Asset editor open on a freshly created asset. A Slate notification toast in the corner reads "New here? The graph already contains a starter scaffold — double-click the Say Line to edit its text." with an "Open Quick Start" link. Caption: "A one-time nudge points you at the Quick Start docs."

---

## Opening the docs from the editor

The dialogue asset editor links straight to this documentation in two places, so the reference is never more than a click away when you are mid-task:

- The **Help** menu → **Open Quick Start Guide**.
- The editor's built-in **Documentation** link (the standard asset-editor help affordance), wired to the same URL.

Both open `https://maystudios.net/maydialogue/docs` in your default browser.

> 📸 **Image placeholder:** `onboarding-help-menu-quickstart.png` — The asset editor Help menu with the Quick Start entry.
> *Setup:* Top of the dialogue asset editor with the **Help** menu open, the **Open Quick Start Guide** item highlighted. A red arrow points to it. Caption: "Documentation is one click away from inside the editor."

---

## The UMG widget footgun — fixed

This used to confuse almost everyone on day one, so it is worth understanding.

**The old confusion:** MayDialogue ships two UI layers — the always-on **Slate debug widget** and an optional **UMG widget** you assign via `DefaultDialogueWidgetClass`. Previously, setting your own UMG widget did **not** turn the Slate debug widget off. The result: people assigned a beautiful custom widget, pressed Play, and saw **both** their widget *and* the plain debug widget stacked on screen — with no obvious reason why.

**The fix:** setting `DefaultDialogueWidgetClass` now **auto-disables the Slate debug widget** (and the settings UI shows a clear hint explaining the relationship). Assign your UMG widget and the debug widget steps aside automatically — no second toggle to remember.

| You set | What shows | Note |
|---|---|---|
| Nothing (default) | Slate debug widget | Works out of the box for testing. |
| `DefaultDialogueWidgetClass` = your UMG widget | Your UMG widget only | Slate debug widget auto-disabled; a hint confirms this. |

{% hint style="warning" %}
If you are on an older build and see two widgets at once, that is the original footgun. Update, or manually uncheck `bUseSlateDialogueWidget` in **Project Settings → Plugins → MayDialogue → Widget**.
{% endhint %}

> 📸 **Image placeholder:** `onboarding-umg-hint.png` — The settings hint shown when a UMG widget is assigned.
> *Setup:* Project Settings → Plugins → MayDialogue → Widget. `DefaultDialogueWidgetClass` is set to a custom widget; below it an info hint reads something like "A UMG widget is assigned — the Slate debug widget is automatically disabled." A red arrow points to the hint. Caption: "Assign a UMG widget and the debug widget steps aside on its own."

---

## Edge cases & gotchas

- **The scaffold is a convenience, not a rule.** If your studio template expects empty assets, delete the scaffold nodes after creation — nothing depends on them.
- **The Dialogue NPC template is a starting point.** It is a plain actor with a participant component; reparent it to your own character class or copy the component onto an existing pawn when you outgrow it.
- **`DefaultDialogue` only matters for the template's default trigger path.** If you start dialogues explicitly via the participant or the library, you can leave it empty and pass the asset at call time — see [Starting a Dialogue](../runtime/starting-dialogues.md).
- **The UMG auto-disable keys off `DefaultDialogueWidgetClass`, not per-call widgets.** If you spawn a dialogue widget yourself through a non-default path, manage the Slate debug widget visibility yourself.
- **The docs links need a network connection** to reach the online documentation.

---

## What's next?

* Build a real, branching dialogue with variables and conditions → [Walkthrough](first-dialogue.md)
* Tune defaults (typewriter speed, advance mode, audio) → [Project Settings](project-settings.md)
* Swap the Slate debug widget for a custom UMG layer → [UI Architecture](../ui/umg-architecture.md)
* Make the dialogue accessible (larger text, screen readers) → [Accessibility](../appendix/accessibility.md)
