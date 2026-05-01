---
description: Search the entire dialogue asset for text, tags, variables, and speakers, then jump to any result with a single click.
---

# Find in Dialogue

Find in Dialogue searches the entire asset for text, tags, variable names, and comments. The result is a clickable list — every hit jumps directly to the relevant node in the graph.

## Opening

- Click the **Find** toolbar button
- or use the shortcut `Ctrl+F`

The **Find Results** tab comes to the foreground.

> 📸 **Image placeholder:** `find-panel-open.png` — Find Results tab in the foreground, search field empty, cursor blinking.
> *Setup:* Asset Editor open, Ctrl+F pressed. Find Results tab jumps to the front. Search field focused, cursor blinking. Empty result list. Red arrow pointing to the search field.

## What is searched?

| Content | Example |
| --- | --- |
| SayLine texts | "Halt! Who are you?" |
| Choice texts | "A friend of the king." |
| PlayerChoice prompt texts | "You reply:" |
| Speaker display names | "Guard", "Narrator" |
| Variable names | "AngerLevel", "HasMet" |
| GameplayTags | `Dialogue.Emotion.Scared`, `Story.Quest.Found` |
| Editor comments | Comment box titles and per-node comments |

The search is **case-insensitive** and uses substring matching: `"cellar"` finds `"The Cellar"` and `"Cellar Room"`.

## Result list

One row per hit:

| Column | Content |
| --- | --- |
| **Match type** | Type of hit: "SayLine-Text", "Choice-Text", "Speaker", "Tag", "Variable", "Comment" |
| **Context** | Hit in context, with the search term highlighted |
| **Node** | Clickable reference — the camera jumps to that node in the graph |

> 📸 **Image placeholder:** `find-results-list.png` — Find Results with four hits for "Cellar".
> *Setup:* "Cellar" searched. Four hits: two SayLine texts (match type "SayLine-Text"), one comment box title (match type "Comment"), one tag `Dialogue.Location.Cellar` (match type "Tag"). All rows with clickable node reference. "Cellar" highlighted in bold in the context column. Red arrow pointing to a node reference.

## Search syntax

The search is deliberately simple: **substring, no regex.** Enter one or more search terms.

Tips:
- Search for **part of a tag**: `"Scared"` finds `Dialogue.Emotion.Scared`
- Search for a **speaker name**: `"Guard"` lists all nodes where the guard speaks
- Search for a **variable name**: `"Anger"` finds nodes that set or check `AngerLevel`
- Search for a **keyword**: `"cellar"` shows all dialogue lines containing that word

> 📸 **Image placeholder:** `find-search-tag.png` — Search field with "Emotion.Scared", result list shows three nodes with that tag.
> *Setup:* "Emotion.Scared" entered in Find. Three hits: two SayLine nodes with EmotionTag `Dialogue.Emotion.Scared`, one Branch node with a Requirement check on the same tag. Match type column shows "Tag". Context shows the full tag string with highlighting.

## Typical use cases

### Unifying text
Search for the old spelling of a term. Click each hit, correct the text in the Details panel.

### Checking tag usage
Search for a GameplayTag (e.g. `Story.Quest.FoundCodex`) — the list shows all nodes that set or check it. Useful for impact analysis before renaming a tag.

### Auditing a variable
Search for the variable name (e.g. `AngerLevel`) — you can immediately see which nodes read it (Requirements) and which write it (SetVariable).

### Speaker audit
Search for a speaker display name — you see all the lines they speak and can check whether their tone is consistent.

### Finding comment boxes
Search for the title of a comment box to jump quickly to a commented section of the graph.

> 📸 **Image placeholder:** `find-variable-audit.png` — Find Results for "AngerLevel", a mix of SayLine SideEffect hits and Branch Requirement hits.
> *Setup:* "AngerLevel" searched. Four hits: two of type "Variable" (SetVariable nodes, context shows "SetVariable: AngerLevel = 1"), two of type "Variable" (CheckDialogueVariable Requirement, context "AngerLevel >= 3"). All four rows have clickable node references.

## Limitations

- **Current asset only.** For a project-wide search across all dialogue assets, use Unreal's built-in Asset Finder or the Reference Viewer.
- **No regex.** Substring matching only.
- **Text fields and tags only.** Numeric property values such as `BlendTime = 1.5` are not found.

{% hint style="info" %}
If you are searching for a tag that appears in many assets, open the assets one at a time and use `Ctrl+F` in each. There is currently no cross-asset search within the MayDialogue editor itself.
{% endhint %}

Next: [Preview Runner →](preview-runner.md)
