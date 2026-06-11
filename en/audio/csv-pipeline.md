---
description: Export your dialogue text to CSV, have it edited or translated, and import it back — the round-trip workflow for proofreading and localization.
---

# CSV Writer Pipeline: Export, Proofread, Import

## When Do I Need This?

When the text in your dialogue assets needs to leave the editor for a while:

- A **proofreader** wants to fix typos and tighten the prose in a spreadsheet, not in the graph editor.
- A **translation agency** needs every line in a single file, one column per language, and sends it back filled in.
- You want a **review snapshot** of all dialogue text in one place — for a script read-through or a word count.

The CSV pipeline round-trips a dialogue asset's text through a plain spreadsheet file. Export writes one row per text field; you (or an external editor) change the cells; import writes the edited source text back into the graph and registers the translated columns so they play at runtime — **without touching the Localization Dashboard.**

This is a **zero-code** workflow. Everything happens through toolbar buttons and right-click actions.

## The Big Picture

```text
[Dialogue Asset]
     │  Export Texts (CSV)
     ▼
[texts.csv]  ── send to proofreader / translator ──▶  [texts_filled.csv]
                                                              │  Import Texts (CSV)
                                                              ▼
            ┌──────────────────────────────────────────────────────────┐
            │ SourceText column  → written back into the graph          │
            │ Culture columns    → UMayDialogueTranslationTable asset    │
            │ (auto-recompile + dirty)                                   │
            └──────────────────────────────────────────────────────────┘
```

> 📸 **Image placeholder:** `csv-pipeline-overview-diagram.png` — Arrow diagram of the export → edit → import round-trip.
> *Setup:* Create a graphic (not an editor screenshot). Left box `[Dialogue Asset]` → arrow labelled "Export Texts (CSV)" → box `[texts.csv]` → arrow labelled "proofread / translate" → box `[texts_filled.csv]` → arrow labelled "Import Texts (CSV)" → split into two boxes `[SourceText → Graph]` and `[Culture columns → Translation Table]`. Horizontal layout, white background, dark-grey arrows.

## No-Code Path

### 1. Export the Texts

Open the dialogue asset. In the asset-editor toolbar, click **Export Texts** (the button reads "Export Texts"; its tooltip mentions CSV). Pick a file location (e.g. `DA_Villager_Intro_texts.csv`). The plugin writes one row per localizable text field.

> 📸 **Image placeholder:** `csv-pipeline-toolbar-export.png` — Asset-editor toolbar with the Export Texts button highlighted.
> *Setup:* Dialogue asset open in the graph editor. Top toolbar visible. Red arrow on the **Export Texts** button (an Export icon). Next to it the **Import Texts** button (an Import icon). Tooltip floating: "Export all localizable text (lines, prompts, choices) to a CSV file for translation."

### 2. Edit in a Spreadsheet

Open the CSV in Excel, LibreOffice Calc, or Google Sheets. The columns are:

| Column | Meaning |
|---|---|
| `NodeGuid` | Stable address of the node. **Do not change.** |
| `NodeType` | `SayLine`, `PlayerChoice`, … (read-only context). |
| `Field` | Which text field on the node — `DialogueText`, `PromptText`, `Choice[2].ChoiceText`, … **Do not change.** |
| `SpeakerTag` | Who speaks the line (context only). |
| `SourceText` | The line in the source culture — **edit this for proofreading.** |
| `en`, `de`, `ja`, … | One column per culture — **fill these for translation.** |

Edit the `SourceText` cells for proofreading; fill the culture columns for translation. Leave `NodeGuid` and `Field` untouched — they are how the importer finds the right node.

> 📸 **Image placeholder:** `csv-pipeline-spreadsheet.png` — The exported CSV open in a spreadsheet with culture columns filled.
> *Setup:* LibreOffice Calc (or Excel) showing a sheet with columns NodeGuid, NodeType, Field, SpeakerTag, SourceText, en, de, ja. Three or four rows of dialogue. The `de` and `ja` columns partly filled with translations. The NodeGuid column shows GUID strings (greyed/narrow). Highlight the SourceText and culture columns as the editable ones.

### 3. Import the Texts Back

Back in the asset editor, click **Import Texts** and pick your edited file. The plugin:

1. Writes the `SourceText` column back into the graph's text fields (matched by `NodeGuid` + `Field`).
2. Collects the non-empty culture cells into a **`UMayDialogueTranslationTable`** asset sitting next to your dialogue (suffix `_Translations`).
3. Recompiles the asset and marks it dirty.

Save the asset. Done — the proofread source text is in the graph, and the translations are live.

> 📸 **Image placeholder:** `csv-pipeline-import-result.png` — Asset editor after import, with the generated translation table asset visible in the content browser.
> *Setup:* Split view. Left: graph editor with a SayLine selected, its `DialogueText` now showing the corrected source line. Right: content browser folder showing the dialogue asset `DA_Villager_Intro` next to the auto-generated `DA_Villager_Intro_Translations` data asset. Red arrow on the new translation table.

### 4. (Optional) Batch Many Assets at Once

In the **content browser**, select one or more dialogue assets, right-click, and use the **Localization → Export Texts (CSV)** / **Import Texts (CSV)** batch actions. This exports/imports without opening each asset editor — ideal for handing off a whole `Content/Dialogue/` folder to a translator in one pass. (Batch export writes one `<AssetName>_Texts.csv` per asset into a folder you pick.)

{% hint style="info" %}
The content-browser batch menu offers **Export** and **Import** only. **Migrate Texts to String Table** is a single-asset action — open the asset and use the editor's menu bar **Localization → Migrate Texts to String Table** (see the next section).
{% endhint %}

> 📸 **Image placeholder:** `csv-pipeline-content-browser-context.png` — Content-browser right-click menu on selected dialogue assets.
> *Setup:* Content browser with three `DA_*` dialogue assets selected (highlighted). Right-click context menu open, showing a **Localization** submenu expanded with entries **Export Texts (CSV)** and **Import Texts (CSV)**.

## How Translations Go Live Without the Dashboard

The imported culture columns are stored as `FPolyglotTextData` entries inside a `UMayDialogueTranslationTable`. That asset **auto-registers on load** (in `PostLoad`): it pushes every entry into the engine's text-localization manager, keyed by the text field's namespace/key. Any `FText` sharing that key — i.e. your dialogue lines — then resolves to the matching per-culture string for the active culture.

The upshot for a beginner: **you do not need to open the Localization Dashboard at all.** Switch the game culture, and the imported translations appear.

{% hint style="info" %}
**This is a convenience layer, not a replacement.** If you already run the UE Localization Dashboard (`.po` / `.locres` workflow), it keeps working unchanged — both feed the same text-localization manager. See [Recipes → Localize Your Dialogues](../recipes/localization.md) for the full Dashboard path.
{% endhint %}

## Migrate to a String Table

For a fully gather-stable, engine-native setup, open the asset and use the editor's menu bar: **Localization → Migrate Texts to String Table**. It creates/updates a per-asset `UStringTable` (suffix `_Texts`) and rebinds every text field to a string-table entry with a stable key (`NodeGuid.FieldId`). After migration the dialogue text is driven by the string table, so the UE Localization Dashboard gathers it reliably.

Use this when you are committing to the Dashboard workflow long-term; use the plain translation table when you just want CSV-supplied translations to play with no further setup.

## API Reference

The editor-side pipeline is `FMayDialogueTextPipeline` (module `MayDialogueEditor`). All entry points are static; the pipeline holds no state.

| Function | Purpose |
|---|---|
| `ExportCSV(Asset, Path, Cultures)` | Write the asset's text fields to a CSV file. One row per field, one column per culture. |
| `ImportCSV(Asset, Path)` | Load a CSV and import it: source text → graph, culture cells → translation table, then recompile + dirty. |
| `BuildCSV(Asset, Cultures)` | Build the CSV string in memory (used by export and tests). |
| `ImportCSVString(Asset, CSVContent)` | Import an in-memory CSV string. Returns true if at least one row matched a node. |
| `MigrateToStringTable(Asset)` | Create/update the `_Texts` `UStringTable` and rebind fields to stable keys. Returns the table. |
| `DiscoverCultures(Asset)` | Compute the culture column set (project loc-target cultures ∪ `VoicePerCulture` keys). |
| `CollectTextFields(Node, OutFields)` | Enumerate the localizable fields on a node (`DialogueText`, `PromptText`, `Choice[i].*`). |
| `MakeStableKey(NodeGuid, FieldId)` | Stable key for a field: `"<NodeGuid>.<FieldId>"`. |
| `MakeNamespace(Asset)` | Generated key namespace: `"MayDialogue.<AssetName>"`. |

**CSV format:** RFC-4180-compliant — fields are quoted when they contain commas, quotes, or newlines; inner quotes are doubled. Column order is `NodeGuid, NodeType, Field, SpeakerTag, SourceText, <culture…>`.

**Runtime asset:** `UMayDialogueTranslationTable` (`Entries: TArray<FPolyglotTextData>`) registers via `RegisterAll()` from `PostLoad()`. It is `BlueprintType`; `RegisterAll` is `BlueprintCallable` for projects that build the table at runtime.

## Edge Cases & Gotchas

{% hint style="warning" %}
**Excel and the semicolon delimiter.** The pipeline writes **comma-separated UTF-8 with a BOM**. The BOM means modern Excel auto-detects the UTF-8 encoding correctly — umlauts and non-Latin scripts survive a plain double-click. The remaining trap is the **delimiter**: on a German/European locale Excel may still expect a **semicolon** rather than a comma and dump every row into one column. If that happens, use **Data → From Text/CSV** and pick **Comma** as the delimiter explicitly. LibreOffice Calc shows the import dialog automatically — leave **UTF-8** and pick **comma**. When saving back, choose "CSV UTF-8" and keep the comma delimiter.
{% endhint %}

**Hand-edited graph text is overwritten on import.** The `SourceText` column is authoritative: importing writes it back into the graph node. If you edited a line directly in the graph *after* exporting, that edit is lost when you import the older CSV. Re-export before importing if the graph changed in the meantime.

**Never reorder or rename the `NodeGuid` and `Field` columns.** They form the address the importer matches on. Rows whose `NodeGuid` no longer resolves to a node (e.g. the node was deleted) are simply skipped. `NodeType` and `SpeakerTag` are context-only — the importer ignores them.

**Import recompiles the asset.** Because the import path mirrors the established compile-on-edit flow, the asset recompiles and is marked dirty right after import. You still have to **save** the asset (and the generated translation/string-table asset) to persist the change. Imports write to the **source** graph node (resolved via `FindSourceNodeByGuid`), so the change survives the next recompile.

**Empty culture cells are ignored.** A blank `de` cell does not create a (broken) empty German translation — it is skipped, and the line falls back per the normal `FText` resolution. Only fill the cells you actually have.

**A choice node produces several rows.** A `PlayerChoice` exports its `PromptText` plus a `Choice[i].ChoiceText` (and `Choice[i].UnavailableReason` when set) per choice. Keep them together — they all share the same `NodeGuid` but differ in `Field`.

## See Also

- [Localize Your Dialogues](../recipes/localization.md) — the full end-to-end localization recipe (Dashboard + CSV + voice per culture).
- [Localization (VoicePerCulture)](localization.md) — a different voice asset per culture.
- [Multilingual Dialogue](../recipes/multilingual-dialogue.md) — the voice-map-focused walkthrough.
