---
description: Wie das MayDialogue-UI aufgebaut ist, wann du Slate nutzt, wann UMG, und wie du Bausteine tauschst.
---

# UI-System

MayDialogue liefert zwei vollständige UI-Implementierungen. Du entscheidest, welche du brauchst.

| | Slate-Debug-Widget | UMG-Komponenten |
|---|---|---|
| Konfiguration | Keine | Blueprint-Subklassen |
| Portraits | Nein | Ja |
| Animationen | Nein | Ja |
| Theming | Fest verdrahtet | Frei |
| Blueprint-erweiterbar | Nein | Ja |
| Wann nutzen | Prototyping, Fallback | Produktion |

> 📸 **Bild-Platzhalter:** `ui-overview-comparison.png` — Zwei Viewport-Screenshots nebeneinander: links das Slate-Debug-Widget (minimalistisch, weißer Text auf dunkel), rechts ein UMG-Widget mit Portrait, animierter Frame und gestylten Buttons.
> *Setup:* PIE starten, gleichen Dialog einmal mit Slate, einmal mit UMG-Widget abspielen. Screenshots in PIE-Viewport machen.

## Wann nutze ich welches?

**Slate** — du möchtest Dialoge schreiben und testen, bevor das UMG-Design steht. Null Konfiguration nötig, sofort spielbar.

**UMG** — du baust das finale Spiel. Du willst Portraits, Animationen, eigene Fonts, eigene Button-Styles und die Freiheit, jeden Baustein gegen deine eigene Blueprint-Klasse zu tauschen.

## Wie ist das UMG-System aufgebaut?

Fünf Widget-Klassen, die du einzeln subclassen und austauschen kannst:

```
UMayDialogueWidget            ← Top-Level (orchestriert Events)
└── UMayDialogueWidget_DialogFrame   ← Container + Animationen
    ├── UMayDialogueWidget_Speaker   ← Name + Portrait + Emotion
    ├── UMayDialogueWidget_Text      ← Typewriter-Text
    ├── UMayDialogueWidget_ChoiceList ← Antwort-Buttons
    │   └── UMayDialogueWidget_ChoiceButton ← einzelner Button
    └── UMayDialogueWidget_SkipButton ← Weiter-Prompt
```

Jede Klasse ist `Abstract, Blueprintable`. Du legst eine Blueprint-Subklasse an, designst das Aussehen im UMG-Designer, und das System verdrahtet alles automatisch via `BindWidget`.

## Wie tausche ich einen Baustein?

Wenn du z.B. das Speaker-Widget gegen dein eigenes tauschen willst:

1. Lege ein neues Blueprint-Widget an — **Parent Class:** `MayDialogueWidget_Speaker`.
2. Baue das Layout im UMG-Designer (PortraitImage, NameText nach Belieben).
3. Implementiere das Event `On Speaker Changed` für eigene Emotion-Logik.
4. Trage die neue Klasse in deinem Top-Level-Widget als `SpeakerWidget`-Slot ein.

Das gilt genauso für alle anderen Komponenten. Du kannst einzelne Bausteine tauschen, ohne den Rest anzufassen.

## Kapitel-Übersicht

* [Slate-Debug-Widget](slate-debug-widget.md) — das sofort einsatzbereite Fallback.
* [UMG-Architektur](umg-architecture.md) — Komposition, Konfigurationspfade, typische Fehler.
* [Dialog Frame](dialog-frame.md) — Container, Animationen, Schnittstelle.
* [Speaker Widget](speaker-widget.md) — Name, Portrait, Emotion-Tags.
* [Text Widget](text-widget.md) — Typewriter-Anbindung, Rich-Text.
* [Choice List & Choice Button](choice-list.md) — Antwort-Buttons, Layout-Optionen.
* [Skip Button](skip-button.md) — Weiter-Prompt, Platform-Hints.
* [Typewriter-Engine](typewriter.md) — Geschwindigkeit, Tags, Skip-to-End.
* [Rich-Text-Tags](rich-text-tags.md) — `<pause>`, `<speed>`, `<shake>`, `<wave>`, `<color>`, `<b>`.
* [Themes & Starterkits](themes.md) — Horror, Visual Novel, RPG.
