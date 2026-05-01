---
description: Creating speakers, choosing colors, setting portraits, and configuring audio overrides.
---

# Speakers Panel

The Speakers panel is the central place where you define all conversation participants of a dialogue asset. Each speaker gets a tag, a display name, a color, and optionally a portrait.

> 📸 **Image placeholder:** `speakers-panel-overview.png` — Speakers panel with three speakers entered.
> *Setup:* Speakers tab active. Three rows visible: "Guard" (dark red, portrait slot filled), "Player" (grey, no portrait), "Narrator" (blue-grey). All fields filled in — tag, display name, portrait thumbnail, color chip.

## Adding a speaker

1. Open the Speakers tab.
2. Click **Add Speaker**.
3. Enter a **Tag** — auto-complete from the central GameplayTags registry.
4. Type a **DisplayName** (the name that appears under the portrait in the UI).
5. Choose a **Portrait** from the asset browser via the dropdown (optional).
6. Set the **Color** via the color picker.
7. Optionally: expand **Audio Overrides** and adjust.

> 📸 **Image placeholder:** `speakers-panel-add-speaker.png` — "Add Speaker" button clicked, a new empty row appears with cursor in the tag field.
> *Setup:* Speakers panel with an existing Guard row. Below it a new empty row with the tag input field active and auto-complete dropdown open. Red arrow on "Add Speaker" button.

## Fields in detail

| Field | Meaning |
| --- | --- |
| **Tag** | GameplayTag for linking to the Participant actor (e.g. `Dialogue.Speaker.Guard`) |
| **DisplayName** | Name displayed in the dialogue UI |
| **Portrait** | Soft reference to a texture (loaded on demand) |
| **Color** | Title bar color of all SayLine nodes for this speaker in the graph |
| **Audio Mode Override** | How voice assets are played (e.g. 2D/3D) |
| **Sound Class Override** | Different SoundClass asset for this speaker |
| **Attenuation Override** | Custom attenuation settings (e.g. for a ghost: softer spatialization) |
| **Volume Multiplier** | Relative volume (1.0 = project default) |
| **Pitch Multiplier** | Relative pitch (1.0 = unchanged) |
| **Babel Profile** | Optional profile for synthesized speech |

## Choosing colors — practical tips

A speaker's color is the **title bar color** of their SayLine nodes in the graph. Well-chosen colors make a 40-node dialogue immediately readable — you can tell at a glance who is speaking.

{% hint style="success" %}
**Tip:** Use the same colors consistently for the same character across all assets. Define the color codes once in a central document or design system.
{% endhint %}

Tried-and-tested color choices:
- Player character: Neutral grey (`#808080`)
- Antagonist: Dark red (`#8B0000`) or dark violet
- Narrator: Dark blue or black
- NPC friend: Muted green or blue-grey

> 📸 **Image placeholder:** `speakers-panel-color-picker.png` — Color picker open, Guard speaker set to dark red.
> *Setup:* Color chip of the Guard entry clicked. UE color picker open, color `#8B0000` set. In the background in the graph you can see a SayLine node immediately showing the new color in its title bar.

## Editing a speaker

All fields are live editable. Change a speaker's **color** — all of that speaker's SayLine nodes in the graph update their title bar immediately.

## Deleting a speaker

Click the **Remove** button in the speaker row. If SayLine nodes still reference the deleted tag, the validator will report a **Missing Speaker** error on the next compile.

## Speaker tag conventions

Recommended format: `Dialogue.Speaker.<Name>` or `Dialogue.Speaker.<Role>`

Examples:
- `Dialogue.Speaker.Player`
- `Dialogue.Speaker.Guard`
- `Dialogue.Speaker.Narrator`
- `Dialogue.Speaker.InnerVoice`

{% hint style="info" %}
The tag must match the `ParticipantTag` of the corresponding NPC actor. Without a match, the dialogue runtime cannot resolve the speaker.
{% endhint %}

## Audio overrides: when to use them

Leave the audio override fields **empty** if the speaker sounds the same as the rest of the project. Only set them when a speaker needs to sound different:

- Ghost: Attenuation and volume at half strength
- Radio noise: Pitch multiplier slightly raised, dedicated SoundClass
- Narrator (off-screen): 2D Audio Mode Override

> 📸 **Image placeholder:** `speakers-panel-audio-overrides.png` — Audio overrides expanded for a ghost speaker.
> *Setup:* Row "Ghost" with the Audio Overrides section expanded. Volume Multiplier at 0.4, Attenuation Override filled, SoundClass Override filled. All other fields empty. For comparison: Guard row below with completely empty audio fields.

## Speaker dropdown in the Details panel

Once you have added speakers in the panel, the `SpeakerTag` field in the node Details shows a **dropdown** instead of a generic tag picker. The dropdown shows:

- Color chip of the speaker
- DisplayName
- Tag (as a tooltip)

This saves time — you don't type tag paths, you pick from a short list.

> 📸 **Image placeholder:** `speakers-dropdown-in-details.png` — Details panel of a SayLine with the speaker dropdown open.
> *Setup:* SayLine node selected. Details panel shows SpeakerTag field as a dropdown, expanded with two entries: Guard (dark red chip, "Guard") and Player (grey chip, "You"). Red arrow on the dropdown.

Next: [Variables Panel →](variables-panel.md)
