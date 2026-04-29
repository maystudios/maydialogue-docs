---
description: Horror-, Visual-Novel- und RPG-Theme — wie du ein Theme wählst, aktivierst und an dein Projekt anpasst.
---

# Themes & Starterkits

{% hint style="warning" %}
**Starter themes (Horror / Visual Novel / RPG) ship as a separate content add-on and are not included in v1.0.** v1.0 ships with the bare `DialogFrame` widget base classes only. See the **Custom Themes** section below to build your own theme from scratch. Pre-built themes will be available in v1.1.
{% endhint %}

MayDialogue liefert das Widget-Framework. Die **visuellen Themes** baust du als Blueprint-Subklassen — jedes ist ein vollständiger Satz Widget-Klassen, der das Aussehen deines Dialogs bestimmt.

> 📸 **Bild-Platzhalter:** `themes-overview-comparison.png` — Drei PIE-Viewport-Screenshots nebeneinander: links Horror-Theme (dunkel, blutrote Akzente), Mitte VN-Theme (helles Panel unten, großes Portrait links), rechts RPG-Theme (pergament-farbenes Banner, kleines Portrait). Gleicher Dialog-Text in allen drei.
> *Setup:* Denselben Dialog dreimal mit unterschiedlichen Widget-Klassen starten. Screenshots in PIE.

## Welche Themes gibt es?

| Theme | Stil | Genre |
|---|---|---|
| **Horror** | Dunkel, blutrote Akzente, grober Rahmen, nervöser Typewriter | Horror, Mystery |
| **Visual Novel** | Großer Portrait-Bereich, zentrierte Text-Box, weiche Animationen | VN, Story-Adventures |
| **RPG** | Klassische Dialog-Box, Name-Plate, Inventar-kompatibles Layout | RPG, Adventure |

Die Widget-Klassen sind subclassbar — du nimmst einen Theme als Ausgangspunkt und passt ihn an.

---

## Theme aktivieren

Setze in den Project Settings:

```ini
[MayDialogue]
DefaultDialogueWidgetClass = /Game/UI/Themes/Horror/WBP_MayDialogue_Horror.WBP_MayDialogue_Horror_C
```

Fertig. Beim nächsten Dialog-Start erscheint das Theme.

> 📸 **Bild-Platzhalter:** `themes-project-settings.png` — Project Settings → MayDialogue. Feld `DefaultDialogueWidgetClass` mit einem ausgefüllten Widget-Pfad, roter Pfeil darauf.
> *Setup:* Project Settings → MayDialogue, Screenshot nur diesen Abschnitt.

---

## Horror-Theme

> 📸 **Bild-Platzhalter:** `theme-horror-ingame.png` — PIE-Viewport, Horror-Theme aktiv. Dunkles Panel unten mit subtilen Rausch-Partikeln im Hintergrund, dünne blutrote Rahmenlinien, Speaker-Name in verwitterter Hand-Lettering-Schrift, Text mit Shake-Effekt auf einem Wort. Einblend-Animation fast fertig (Flicker-Fade).
> *Setup:* Horror-Theme-Widget setzen, Dialog mit `<shake>`-Tag und Emotion-Tag "Dialogue.Emotion.Scared" starten. Screenshot kurz nach dem Einblenden.

**Dialog Frame**
- Hintergrund: pitch-black, subtiles Noise-Overlay.
- Rahmen: dünne blutrote Linien, leicht asymmetrisch.
- Einblend-Animation: schneller Flicker-Fade (200 ms).
- Ausblend-Animation: Glitch-Effekt (Pixel verschieben sich kurz, Fade-Out).

**Speaker**
- Portrait-Frame: Cracked-Glass-Look.
- Name-Font: Hand-Lettering oder leicht schief gesetzt.
- Emotion "Scared": Portrait wechselt + Shake-Animation.

**Text**
- Typewriter: leicht unregelmäßig (Variation mit `<speed>`-Tags im Inhalt).
- Font: Serif, leicht verwittert.
- Shake bei emotionalen Wörtern empfohlen: `<shake>Tot.</shake>`.

**Choice-Buttons**
- Hover: roter Glow, kurzes Zucken.
- Disabled: fast unsichtbar, Tooltip erscheint verzögert.

> 📸 **Bild-Platzhalter:** `theme-horror-choice-hover.png` — PIE-Viewport, Horror-Theme, PlayerChoice aktiv. Button 1 gehovered: roter Glow um den Button-Bereich. Button 2 normal. Button 3 disabled (kaum sichtbar). Maus-Cursor sichtbar.
> *Setup:* Horror-Theme, PlayerChoice-Node mit 3 Choices (1 unavailable), Maus über Button 1 hovern, Screenshot.

---

## Visual-Novel-Theme

> 📸 **Bild-Platzhalter:** `theme-vn-ingame.png` — PIE-Viewport, VN-Theme aktiv. Helles, halbtransparentes Panel unten zentriert (70% Breite). Portrait links groß (256×256), daneben Speaker-Name als Badge über dem Text. Sanfte Slide-Up-Animation abgeschlossen. Drei zentrierte Choice-Zeilen.
> *Setup:* VN-Theme-Widget setzen, Dialog mit Portrait und PlayerChoice starten. Screenshot nach abgeschlossener Einblende.

**Dialog Frame**
- Hintergrund: 70% Deckkraft, Pergament-Farbe, 12 px Abrundung.
- Position: unten zentriert, 70% Bildschirmbreite.
- Einblend-Animation: Slide-Up + 300 ms Ease-Out.

**Speaker**
- Portrait: 256×256, links positioniert.
- Emotion-Tags steuern Portrait-Wechsel via DataTable (Emotion → Texture).
- Name-Badge: über dem Text-Bereich, nicht im Speaker-Widget selbst.

**Text**
- Typewriter mittlere Geschwindigkeit.
- Font: Sans-Serif, gut lesbar.

**Choice-Buttons**
- Zentrierte Hover-Zeilen.
- Optional: Tasten-Hinweis (Q, W, E, R) am Button-Ende.

> 📸 **Bild-Platzhalter:** `theme-vn-choice-layout.png` — PIE-Viewport, VN-Theme, drei zentrierte Choice-Buttons. Jeder Button hat am rechten Rand einen Tasten-Hinweis ("Q", "W", "E"). Button 2 gehovered (Highlight-Farbe). Keine Disabled-Buttons in diesem Screenshot.
> *Setup:* VN-Theme, PlayerChoice mit 3 Choices, Tasten-Hints in WBP_ChoiceButton integriert, Screenshot.

---

## RPG-Theme

> 📸 **Bild-Platzhalter:** `theme-rpg-ingame.png` — PIE-Viewport, RPG-Theme aktiv. Pergament-farbenes Banner unten (80% Breite), verzierter Rand, Name-Plate separat über dem Frame, kleines Portrait links (64×64), Dialog-Text in Serif-Font. Drei Choice-Buttons mit Icons links (Schild-Icon, Sprech-Icon, Lauf-Icon).
> *Setup:* RPG-Theme-Widget setzen, Dialog mit kleinem Portrait und 3 Choices mit unterschiedlichen Icons starten.

**Dialog Frame**
- Hintergrund: semi-opak, brauner Banner mit verziertem Border.
- Position: unten, 80% Bildschirmbreite.

**Speaker**
- Name-Plate: separat über dem Frame, nicht im Frame selbst.
- Portrait: klein (64×64), links.

**Text**
- Typewriter schnell.
- Font: Serif oder fantasievolle Kursive.

**Choice-Buttons**
- Icons neben dem Choice-Text (Kampf-Icon, Sprach-Icon, Magie-Icon).

---

## Eigenes Theme bauen

Ein Theme besteht aus Blueprint-Subklassen aller Widget-Klassen plus einem Top-Level-Widget das sie composited.

**Schritt 1 — Subklassen anlegen**

Erstelle Blueprint-Subklassen für alle Bausteine:

| Blueprint | Parent Class |
|---|---|
| `WBP_Theme_DialogFrame` | `UMayDialogueWidget_DialogFrame` |
| `WBP_Theme_Speaker` | `UMayDialogueWidget_Speaker` |
| `WBP_Theme_Text` | `UMayDialogueWidget_Text` |
| `WBP_Theme_ChoiceList` | `UMayDialogueWidget_ChoiceList` |
| `WBP_Theme_ChoiceButton` | `UMayDialogueWidget_ChoiceButton` |
| `WBP_Theme_SkipButton` | `UMayDialogueWidget_SkipButton` |

**Schritt 2 — Assets anlegen**

- Background-Texturen (Rauschen, Pergament, Banner).
- Fonts (Serif, Hand-Lettering, Sans-Serif).
- SoundCues für UI-SFX (Hover, Click, Dialogue-Open).

**Schritt 3 — Top-Level-Widget compositen**

Lege `WBP_MayDialogue_Theme` an (Parent: `UMayDialogueWidget`). Füge die Sub-Widgets mit korrekten Namen ein und binde sie via BindWidget.

**Schritt 4 — Aktivieren**

```ini
DefaultDialogueWidgetClass = /Game/UI/Themes/YourTheme/WBP_MayDialogue_Theme.WBP_MayDialogue_Theme_C
```

> 📸 **Bild-Platzhalter:** `theme-custom-before-after.png` — Vorher/Nachher-Split: Links Standard-Slate-Debug-Widget (minimalistisch). Rechts: fertiges Custom-Theme mit allen visuellen Elementen aktiv. Gleicher Dialog-Text, gleicher Sprecher.
> *Setup:* Denselben Dialog einmal mit Slate, einmal mit fertigem Custom-Theme starten. Screenshots nebeneinander.

## Mehrere Themes im selben Projekt

Für unterschiedliche Dialog-Kontexte (Haupt-Story vs. Tutorial vs. Cutscene) tauschst du das Widget vor dem Dialog-Start:

```cpp
// Vor dem Dialog-Start in deinem Game-Code:
// Project Settings programmatisch überschreiben oder
// über einen eigenen Manager die Widget-Klasse pro Kontext setzen.
```

{% hint style="info" %}
Ein dedizierter API-Hook für Widget-Swap pro Dialog ist in Planung. Aktuell: Widget-Klasse in den Settings vor dem Dialog-Start setzen oder einen eigenen Widget-Manager bauen, der das Widget beim Dialog-Start tauscht.
{% endhint %}

Ende UI-Kapitel. Weiter: [Audio-System](../audio/README.md).
