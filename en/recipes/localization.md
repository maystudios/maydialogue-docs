---
description: The complete localization workflow — FText source text, the Loc-Audit hint, String Tables, the UE Localization Dashboard, per-culture voice, and in-editor culture preview.
---

# Localize Your Dialogues

## Scenario

Your game ships in several languages. Every dialogue line needs translated text, and ideally a matching voice recording per language. This recipe is the **end-to-end localization path**: from authoring gather-stable source text, through translation (Dashboard or CSV), to per-culture voice and an in-editor preview that switches text and voice together.

It ties together three tools that each have their own page — here they are sequenced into one workflow.

## What You Will Learn

- Why dialogue text must be `FText` (not `FString`) and how keys make it gather-stable.
- Read and clear the **Loc-Audit** validator hint.
- Run **Migrate Texts to String Table** for reliable gathering.
- Drive the UE **Localization Dashboard**: gather → translate → compile.
- Assign **per-culture voice** via `VoicePerCulture`.
- Preview a culture in the editor — text **and** voice switch together.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- The cultures you want registered in **Project Settings → Internationalization** (e.g. native `en`, plus `de`, `ja`).

## Step by Step

### 1. Author Text as FText

Every dialogue text field — `DialogueText` on SayLine, `PromptText` and `ChoiceText` on PlayerChoice — is already an `FText`. That is the prerequisite for localization: `FText` carries a namespace/key, and the UE gather pipeline only collects `FText`, never raw strings. Just type your source-culture lines normally; you do not need to do anything special at authoring time.

{% hint style="info" %}
**Speaker display names too.** Keep the speaker `DisplayName` on the asset's Speakers panel as `FText` so it gets gathered alongside the lines. A `FString` display name will not be translated.
{% endhint %}

### 2. Read the Loc-Audit Hint

Open the dialogue asset and compile. The validator runs a **Loc-Audit** check: for any non-empty SayLine / PlayerChoice / Choice text that is not backed by a String Table and has no stable localization key, it emits a blue **Hint** on the node. The hint is informational — the dialogue still compiles — but it flags text that may not gather reliably for a localized build.

The hint summarizes the affected fields and points you at the fix: run **Migrate Texts to String Table**, or otherwise give the text a stable key.

> 📸 **Image placeholder:** `localization-loc-audit-hint.png` — A SayLine node showing the blue Loc-Audit hint in the validation panel.
> *Setup:* Dialogue asset open. A SayLine node in the graph. The validation/message panel (bottom) shows a blue Hint entry: "Loc-Audit: 'DialogueText' has no stable localization key — run Migrate Texts to String Table". The node itself has a muted blue indicator (not a red error border). Red arrow on the hint row.

### 3. Migrate to a String Table

Open the asset and use the editor's menu bar: **Localization → Migrate Texts to String Table** (it lives in the asset editor's menu, not the toolbar, and is single-asset only). The plugin creates/updates a `UStringTable` next to the asset (suffix `_Texts`) and rebinds every text field to a string-table entry with a stable key (`NodeGuid.FieldId`). The Loc-Audit hint clears. Now the text is engine-native and gathers reliably.

> 📸 **Image placeholder:** `localization-migrate-string-table.png` — The menu action and the resulting `_Texts` string table asset.
> *Setup:* Split view. Left: asset-editor menu bar open at the **Localization** section with **Migrate Texts to String Table** highlighted (red arrow). Right: content browser showing `DA_Villager_Intro` next to the generated `DA_Villager_Intro_Texts` string-table asset.

{% hint style="info" %}
**Two valid paths.** Migrating to a String Table is the gather-stable, Dashboard-friendly route. If you instead use the [CSV pipeline](../audio/csv-pipeline.md), its translation table also makes translations live without the Dashboard. Pick the one that fits your team — they can coexist.
{% endhint %}

### 4. Gather, Translate, Compile (Localization Dashboard)

**Window → Localization Dashboard.** Select your project's localization target (the entry that includes the plugin's gathered text — by default the **Game** target, which gathers `Content/` including your dialogue String Tables).

1. **Gather Text** — collects all `FText` / String Table entries, including your migrated dialogue text.
2. **Add the target cultures** (`de`, `ja`, …) if not present.
3. **Edit translations** — open the culture editor and enter the translated strings.
4. **Compile Text** — writes the `.locres` files the runtime reads.

> 📸 **Image placeholder:** `localization-dashboard-gather.png` — Localization Dashboard after a successful gather + compile.
> *Setup:* Window → Localization Dashboard open. The Game target row visible with a green status. Culture list showing en/de/ja with translation-progress bars. The Gather, Compile, and Edit buttons visible. Red arrow on the Compile button.

### 5. Assign Per-Culture Voice

Translated text is half the job; for voiced games, assign a recording per language. On each SayLine, fill the `VoicePerCulture` map: key = BCP-47 culture code (`en`, `de`, `ja`), value = the `USoundBase` for that language. The default `DialogueVoice` slot is the fallback when no culture key matches. Full details: [Localization (VoicePerCulture)](../audio/localization.md).

> 📸 **Image placeholder:** `localization-voiceperculture-recipe.png` — SayLine details with the VoicePerCulture map filled for three cultures.
> *Setup:* SayLine node selected. Details "SayLine → Audio": `DialogueVoice` = `SW_Villager_Line01` (fallback), and the `VoicePerCulture` map with `en → SW_..._EN`, `de → SW_..._DE`, `ja → SW_..._JA`. Red arrow on the map.

### 6. Preview a Culture in the Editor

Open the [Preview Runner](../editor/preview-runner.md) and switch the culture dropdown. **Text and voice switch together**: you immediately see the translated line and hear the matching `VoicePerCulture` recording, without restarting the editor. This is the fast loop for checking that a translation reads well at the right length and that the right take plays.

> 📸 **Image placeholder:** `localization-preview-culture-switch.png` — Preview Runner with the culture dropdown set to `de`, showing the German line.
> *Setup:* Preview Runner panel. Culture dropdown at the top set to "de" (open or showing selection). The active line displayed in German. A small speaker icon indicating the German voice take is playing. Caption: "Text + voice switch together."

## The Workflow at a Glance

```text
1. Author FText source lines        (just type them)
2. Compile → read Loc-Audit hint    (flags gather-unstable text)
3. Migrate Texts to String Table    (gather-stable keys)
4. Localization Dashboard            (gather → translate → compile)
5. Fill VoicePerCulture per SayLine  (audio per language)
6. Preview Runner culture dropdown   (text + voice switch together)
```

## Variations / Going Further

- **CSV instead of (or alongside) the Dashboard**: hand the whole text set to a translator as a spreadsheet — see [CSV Writer Pipeline](../audio/csv-pipeline.md). The imported translation table plays without any Dashboard step.
- **Partial voicing**: fill only the `en` voice and leave others empty; the Babel synth (if enabled) fills missing-language audio so you can ship-test before all takes exist.
- **Switch culture at runtime**: `Set Current Culture` (Blueprint) or `FInternationalization::Get().SetCurrentCulture(...)` (C++); the next dialogue start uses the new culture for both text and voice.
- **CJK / RTL scripts**: a translation problem only if the UMG font lacks the glyphs — give the dialogue widget a font with the needed script or a composite-font fallback chain.

## Troubleshooting

**Text stays in the source language after switching culture.**
The localization target was not **compiled** after gathering, so no `.locres` exists. Re-run Compile Text. Also confirm the dialogue text was migrated to a String Table or otherwise has a stable key — a gather-unstable literal (flagged by Loc-Audit) may not be in the `.locres` at all.

**Loc-Audit hint will not clear.**
The text field still has no stable key. Run **Migrate Texts to String Table** on the asset. Empty and culture-invariant texts are intentionally skipped by the audit and never show a hint.

**Translated text shows, but the old voice plays.**
The `VoicePerCulture` map has no entry for the active culture, so it falls back to `DialogueVoice`. Add the culture key, or accept the fallback. Region variants (`de-AT`) fall back to the base code (`de`) automatically.

**Gather does not find the dialogue text.**
The text is a raw `FString` somewhere, or it was placed as a Blueprint literal rather than on the asset. Only `FText` fields on the dialogue graph (and String Tables) are gathered. Migrate to a String Table to be safe.

## See Also

- [CSV Writer Pipeline](../audio/csv-pipeline.md) — spreadsheet round-trip for proofreading and translation.
- [Localization (VoicePerCulture)](../audio/localization.md) — the per-culture voice map in depth.
- [Multilingual Dialogue](multilingual-dialogue.md) — a voice-map-focused companion recipe.
- [Preview Runner](../editor/preview-runner.md) — the culture-switching preview.
