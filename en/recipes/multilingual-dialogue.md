---
description: Localize text via FText Gather and assign voice files per culture – the complete L10n workflow.
---

# Multilingual Dialogue

## Scenario

Your game ships in German, English, and Japanese. An NPC says two lines, each with its own voice recording per language. This recipe walks you through the complete process: configuring cultures, gathering text, filling the voice map, and switching languages at runtime.

## What You Will Learn

- Register cultures in the UE project.
- Run FText Gather for dialogue assets.
- Fill the `VoicePerCulture` map on the SayLine node.
- Switch language at runtime via Blueprint and C++.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- UE Localization Target created for the project.
- Voice files imported as `USoundBase`: `SW_Villager_Line01_DE`, `_EN`, `_JA`.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Villager – "Good day, stranger."  VoicePerCulture: {de, en, ja}]
   │
   ▼
[SayLine: Villager – "Be on your guard."    VoicePerCulture: {de, en, ja}]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `multilingual-dialogue-graph-overview.png` — Asset Editor with a SayLine, VoicePerCulture map expanded.
> *Setup:* Asset `DA_Villager_Intro` open. SayLine selected, Details panel on the right shows `DialogueText = "Good day, stranger."` and below it the `VoicePerCulture` map with three rows: `de → SW_Villager_Line01_DE`, `en → SW_Villager_Line01_EN`, `ja → SW_Villager_Line01_JA`.

## Step by Step

### 1. Register Cultures in the Project

**Edit → Project Settings → Internationalization**:
- `Native Culture`: `de`
- `Cultures to Stage`: `de`, `en`, `ja`

### 2. Create the Dialogue Asset and Enter Text

Asset: `DA_Villager_Intro`. SayLine `DialogueText` in the native culture (German): *"Guten Tag, Fremder."*

`FText` automatically stores a text key on asset save — the gather system uses this later.

### 3. Fill the Voice-per-Culture Map

On the SayLine node under `DialogueVoice` → map with three entries:

| Key | Value |
|-----|-------|
| `de` | `SW_Villager_Line01_DE` |
| `en` | `SW_Villager_Line01_EN` |
| `ja` | `SW_Villager_Line01_JA` |

For the fallback key `""` (empty string): used when the current culture is not in the map. Leave empty if Babel synthesis should be active.

### 4. Run the Text Gather

**Window → Localization Dashboard** → Gather Text → Compile Text → Edit (for entering translations).

Via CLI:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Binaries/Win64/UnrealEditor-Cmd.exe" ^
  "C:/UnrealEngine/VHS/VHS.uproject" ^
  -run=GatherText -Target=Game
```

> 📸 **Image placeholder:** `multilingual-dialogue-localization-dashboard.png` — Localization Dashboard after a successful gather.
> *Setup:* Window → Localization Dashboard open. Game target visible, status column shows green checkmark after Gather+Compile. Culture list with de/en/ja and progress bars.

### 5. Enter the Translations

In the Culture Editor (Compile → Edit), enter the English and Japanese texts. MayDialogue uses the active UE culture when the dialogue starts for text display.

### 6. Runtime Language Switch

**Blueprint:**

```text
[Set Current Culture]
   └─ Culture: "en"
```

**C++:**

```cpp
FInternationalization::Get().SetCurrentCulture(TEXT("en"));
```

On the next dialogue start, the plugin uses the new culture for text and voice selection.

## Voice Resolution

```text
ExecuteNode SayLine
   → Get current culture (FInternationalization)
   → Key in VoicePerCulture?
       ├─ Yes → Play voice
       └─ No → Fallback key ("") set?
                   ├─ Yes → Play fallback voice
                   └─ No → Babel active? → Synthesis / silence
```

## Recommended Folder Structure

```text
Content/
  Dialogue/
    Villager/
      DA_Villager_Intro.uasset
      Voice/
        de/SW_Villager_Line01.wav
        en/SW_Villager_Line01.wav
        ja/SW_Villager_Line01.wav
```

Via culture folders you can selectively include or exclude culture sets during packaging in Perforce.

## Mix: Only Partially Voiced

| Scenario | Setup |
|----------|-------|
| Only EN voiced, others text only | Fill only the `en` key |
| DE + EN voiced, JA text only | Fill `de` + `en`, leave `ja` empty |
| All cultures via Babel | Leave VoiceMap empty, enable Babel in Project Settings |

> 📸 **Image placeholder:** `multilingual-dialogue-ingame-preview.png` — Preview Runner with active dialogue in English.
> *Setup:* Preview Runner panel with English culture. Text box shows `"Good day, stranger."`, speaker name `Villager`. Note: culture must be set before starting the Preview.

## Variations / Going Further

- Enable Babel synthesis for missing cultures → [Audio → Babel System](../audio/babel-system.md).
- Also maintain the speaker `DisplayName` as `FText` (not `FString`) so it gets gathered too.
- Culture-specific emotions: EmotionTags are GameplayTags (not localizable), so they are language-independent.

## Troubleshooting

**English voice plays, but text stays in German.**
Localization target not Compiled after the Gather. `.locres` file missing. Re-run Compile Text.

**Voice fallback doesn't trigger.**
Empty `""` key not filled. Either enable Babel (`bEnableBabelVoice` in Project Settings) or fill the key explicitly.

**Gather doesn't find the text.**
Text was placed as a Blueprint literal into a variable field instead of directly on the asset. Only `FText` properties in dialogue graphs are gathered.

**Japanese script not visible.**
This is not a MayDialogue problem — the UMG widget needs a font with CJK support or a font fallback chain (UE Composite Font Assets).
