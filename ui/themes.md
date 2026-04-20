# Themes & Starterkits

MayDialogue liefert das Widget-Framework. Die **visuellen Themes** sind als Starterkits angedacht – Blueprint-Widgets, die deine Grundrichtung setzen.

## Geplante Themes

| Theme | Stil | Ziel-Genre |
| --- | --- | --- |
| **Horror** | Dunkler Untergrund, blutrote Akzente, grob-pixeliger Rahmen, nervöser Typewriter | Horror, Mystery |
| **Visual Novel** | Großer Portrait-Bereich, zentrierte Text-Box unten, weiche Open-Animation | VN, Story-Adventures |
| **RPG** | Klassische Dialog-Box mit Name-Plate, Inventar-kompatibles Layout | RPG, Adventure |

{% hint style="info" %}
**Status**: Die Themes sind als Roadmap-Item geplant (Backlog-Item 6). Die aktuelle Plugin-Version liefert die Widget-Klassen; Blueprint-Skins werden nachgeliefert.
{% endhint %}

## Eigenes Theme bauen

So gehst du vor:

1. **Blueprint-Subklassen** der fünf Widget-Klassen anlegen:
   * `WBP_Theme_DialogFrame` (von `UMayDialogueWidget_DialogFrame`).
   * `WBP_Theme_Speaker` (von `UMayDialogueWidget_Speaker`).
   * `WBP_Theme_Text` (von `UMayDialogueWidget_Text`).
   * `WBP_Theme_ChoiceList` (von `UMayDialogueWidget_ChoiceList`).
   * `WBP_Theme_ChoiceButton` (von `UMayDialogueWidget_ChoiceButton`).
   * `WBP_Theme_SkipButton` (von `UMayDialogueWidget_SkipButton`).
2. **Compose** zu einem `WBP_MayDialogue_Theme`-Top-Level-Widget, das die Sub-Widgets per BindWidget hält.
3. **Theme-Assets** anlegen: Background-Bilder, Fonts, SoundCues für UI-SFX.
4. **Styling** im Designer jedes Widgets anwenden.

## Horror-Theme-Inspiration

**Dialog-Frame**:

* Background: pitch-black mit subtilem rauschenden Rauschen.
* Borders: dünne blut-rote Linien, etwas asymmetrisch.
* Open-Animation: schneller Flicker-Fade-In (200 ms).
* Close-Animation: Glitch-Effekt (Pixel verschieben sich kurz, dann Fade-Out).

**Speaker**:

* Portrait-Frame: cracked glass look.
* Name-Font: hand-lettering oder leicht schief gesetzt.

**Text**:

* Typewriter-Geschwindigkeit: etwas unregelmäßig (Variation mit `<speed>`-Tags).
* Default-Font: Serif, leicht verwittert.
* Shake bei emotionalen Wörtern (`<shake>Tot.</shake>`).

**Choice-Button**:

* Hover: roter Glow, kurzes Zucken.
* Disabled: fast unsichtbar, Tooltip erscheint verzögert.

## Visual-Novel-Theme-Inspiration

**Dialog-Frame**:

* Background: 70%-deckende Pergament-Farb-Box, Abrundung 12 px.
* Position: unten zentriert, 70% Bildschirmbreite.
* Open-Animation: Slide-Up + 300 ms Ease-Out.

**Speaker**:

* Portrait-Größe: groß (256×256), links positioniert.
* Emotion-Tags blenden Portrait-Variante um (z.B. über dedizierte `DataTable` Emotion → Texture).

**Text**:

* Typewriter mittlere Geschwindigkeit.
* Default-Font: Sans-Serif, lesbar.
* Name-Badge oberhalb des Textes.

**Choice-Button**:

* Zentriert als Hover-Zeilen.
* Key-Binding-Hint (Q, W, E, R).

## RPG-Theme-Inspiration

**Dialog-Frame**:

* Background: semi-opaque brauner Banner mit verziertem Border.
* Position: unten, 80% Bildschirmbreite.

**Speaker**:

* Name-Plate separat über dem Frame.
* Portrait klein (64×64), links.

**Text**:

* Typewriter schnell.
* Font: Serif oder fantasievolle Kursive.

**Choice-Button**:

* Icons neben Choice-Text (Kampf-Icon, Sprach-Icon, Magie-Icon).
* Inventar-Buttons unten (wenn relevant).

## Theme-Zuweisung in Project-Settings

```
[Project Settings → MayDialogue]
DefaultDialogueWidgetClass = /Game/UI/Themes/Horror/WBP_MayDialogue_Horror.WBP_MayDialogue_Horror_C
```

Fertig. Beim nächsten Dialog-Start taucht dein Theme auf.

## Mehrere Themes im selben Projekt

Manche Spiele haben unterschiedliche Dialog-Kontexte (Main-Story vs. Tutorial vs. Minigame). Du kannst pro Asset oder pro Szene das Widget dynamisch wechseln:

```cpp
Sub->SetCustomWidgetClass(TutorialWidgetClass);  // API geplant; aktuell workaround per Project-Setting-Swap
```

{% hint style="warning" %}
Ein dediziertes API-Hook für Widget-Swap ist in der Roadmap. Bis dahin: Widget-Klasse pro Szene durch Code tauschen oder vor Dialog-Start in den Settings überschreiben.
{% endhint %}

Ende UI-Kapitel. Weiter: [Audio-System →](../audio/README.md).
