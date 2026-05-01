---
description: A separate voice asset for each language — how the VoicePerCulture map works and how the system resolves at runtime.
---

# Localization (VoicePerCulture)

## When Do I Need This?

When your game ships in multiple languages and you have separate voice recordings for each:

- English voice for `en`, German for `de`, Japanese for `ja`
- At runtime the correct recording plays automatically — no Blueprint logic required

MayDialogue combines this with UE's standard text localization: the **text** of a SayLine is translated via the normal UE localization dashboard, and the **voice** is resolved via the `DialogueVoice` map on the SayLine.

## The VoicePerCulture Map

Every SayLine has a map:

```text
DialogueVoice:
  "en" → VO_EN_Hello
  "de" → VO_DE_Hallo
  "ja" → VO_JA_Konnichiwa
```

Key: BCP-47 culture code. Value: the `USoundBase` asset for that culture.

> 📸 **Image placeholder:** `localization-voiceperculture-map.png` — SayLine details panel with the DialogueVoice map filled in.
> *Setup:* Select a SayLine node in the dialogue editor. Details panel on the right: "Audio" section expanded, `DialogueVoice` map visible with three entries: key `en` → asset `VO_EN_Hello`, key `de` → asset `VO_DE_Hallo`, key `ja` → asset `VO_JA_Konnichiwa`. Red arrow pointing to the map entries.

## How to Set Entries

1. Select the SayLine node in the graph
2. Details panel → "Audio" section → `DialogueVoice`
3. `+` button → enter key (e.g. `"de"`) → fill the asset slot with a voice asset
4. Repeat for each language

> 📸 **Image placeholder:** `localization-add-entry.png` — DialogueVoice map with the "+" button and a half-filled new entry.
> *Setup:* Same view as above, but focus on the `+` button and a newly created, still empty key-value entry. Key field is active (cursor visible), value slot still empty. Shows the add workflow.

## Runtime Resolution Logic

MayDialogue queries UE's locale API for the current culture and resolves as follows:

1. Look up `DialogueVoice[current culture]` — e.g. `"de-AT"`
2. Not found? → strip the region, look up `"de"`
3. Not found? → look up the default key (typically `"en"` or an empty string)
4. Still not found? → no voice asset → Babel or silence

```text
Requested culture: "de-AT"
  → Lookup "de-AT" → not found
  → Lookup "de"    → found: VO_DE_Hallo ✓
```

## Common Culture Codes

| Code | Language |
|---|---|
| `en` | English |
| `de` | German |
| `fr` | French |
| `ja` | Japanese |
| `zh-Hans` | Chinese (Simplified) |
| `es-419` | Spanish (Latin America) |
| `pt-BR` | Portuguese (Brazil) |

UE uses BCP-47 codes. Regional variants (`de-AT`, `en-GB`) automatically fall back to the base code.

## Combining with Text Localization

The text (`DialogueText`) on every SayLine is an `FText` and goes through UE's standard localization pipeline:

- Localization Dashboard gathers all texts
- Export as `.po` files for translators
- Import translated `.po` files
- Runtime culture switch translates the text immediately

Combined with the `DialogueVoice` map, you get both the correct text and the correct voice for each language.

## Switching Culture in the Preview Runner

The [Preview Runner](../editor/preview-runner.md) has a culture dropdown. One click switches the active culture — you immediately hear the correct voice asset and see the translated text, without restarting the editor.

> 📸 **Image placeholder:** `localization-preview-runner-culture-dropdown.png` — Preview Runner with the culture dropdown open, active culture "de".
> *Setup:* Preview Runner panel in the editor (bottom or as a separate window). A dropdown menu at the top of the panel for culture selection. Dropdown open, options: en, de, ja, fr. "de" is highlighted. The active dialogue text is visible in German below.

## Babel as a Fallback for Missing Languages

If a SayLine has no voice asset for the current culture (and no default key is present), Babel activates automatically — provided `bEnableBabelVoice` is enabled in the Project Settings. Babel synthesises from the text string, regardless of language.

{% hint style="info" %}
**Babel as a development placeholder:** In early project phases it is enough to set only `"en"` voice assets. For all other cultures, Babel plays. This lets you test localization flows without having all recordings finished.
{% endhint %}

## Notes on Asset Management

{% hint style="warning" %}
Voice assets are stored as **hard references**. They are packaged when the package is cooked. For large voice libraries: enable the `Streaming` flag in the SoundWave properties so the memory footprint at level start remains controlled.
{% endhint %}
