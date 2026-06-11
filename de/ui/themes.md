---
description: Horror-, Visual-Novel- und RPG-Theme — wie du ein Theme wählst, aktivierst und an dein Projekt anpasst.
---

# Themes & Starterkits

{% hint style="success" %}
**Drei fertige Themes liefert das Plugin mit** — unter `/MayDialogue/Themes`: Horror (**Pale Static**), Visual Novel (**Daybreak**) und RPG (**Gilded Slate**) — dazu ein neutrales `WBP_DialogueTheme_Default` unter `/MayDialogue/Samples/UI`. Jede Sample-Map setzt ihr Theme automatisch über einen `AMayDialogueThemeSetter`-Actor, sodass du jedes Sample-Level starten und sofort ein fertiges Theme sehen kannst. Nimm eins davon als Ausgangspunkt und subklasse es, oder bau dir mit dem Abschnitt **Eigenes Theme bauen** unten ein eigenes von Grund auf.
{% endhint %}

MayDialogue liefert das Widget-Framework und obendrauf fertige Themes. Die **visuellen Themes** sind Blueprint-Subklassen — jedes ist ein vollständiger Satz Widget-Klassen, der das Aussehen deines Dialogs bestimmt.

> 📸 **Bild-Platzhalter:** `themes-overview-comparison.png` — Drei PIE-Viewport-Screenshots nebeneinander: links Horror-Theme (fast schwarzes Panel, Typewriter-Schrift im Nameplate), Mitte VN-Theme (tiefviolettes Panel, Name-Pill, helle Sans-Serif), rechts RPG-Theme (dunkles Slate-Panel mit Gilt-Rahmen, Cinzel-Nameplate, elfenbeinfarbener Text). Gleicher Dialog-Text in allen drei.
> *Setup:* Denselben Dialog dreimal mit unterschiedlichen Widget-Klassen starten (je Sample-Map einmal anspielen — der ThemeSetter setzt das Theme automatisch). Screenshots in PIE.

## Welche Themes gibt es?

Drei Themes liegen fertig nutzbar unter `/MayDialogue/Themes`. Jedes ist ein vollständiger Satz subklassierter Widget-Klassen plus ein Top-Level-Composite (`WBP_MayDlg_Theme_Horror` / `_VN` / `_RPG`):

| Theme | Composite-Widget | Stil | Genre |
|---|---|---|---|
| **Horror** — *Pale Static* | `WBP_MayDlg_Theme_Horror` | Fast schwarzes Panel, fahler Text, Typewriter-Schrift (Special Elite) im Nameplate, CRT-Flicker beim Einblenden | Horror, Mystery |
| **Visual Novel** — *Daybreak* | `WBP_MayDlg_Theme_VN` | Tiefviolettes, breites Panel, Name-Pill (Quicksand), Nunito-Sans-Fließtext, Choices schweben mittig im Bild | VN, Story-Adventures |
| **RPG** — *Gilded Slate* | `WBP_MayDlg_Theme_RPG` | Dunkles Slate-Panel mit Gilt-Rahmen, Cinzel-Nameplate, elfenbeinfarbener Text, Lock-Icon auf gesperrten Choices | RPG, Adventure |

Ein neutrales `WBP_DialogueTheme_Default` (unter `/MayDialogue/Samples/UI`) ist die schlichte Basis, die genutzt wird, wenn kein Theme gesetzt ist. Die Widget-Klassen sind subclassbar — du nimmst ein beliebiges Theme als Ausgangspunkt und passt es an.

---

## Theme aktivieren

Für einen projektweiten Default setze in den Project Settings:

```ini
[MayDialogue]
DefaultDialogueWidgetClass = /MayDialogue/Themes/Horror/Widgets/WBP_MayDlg_Theme_Horror.WBP_MayDlg_Theme_Horror_C
```

Fertig. Beim nächsten Dialog-Start erscheint das Theme. Um ein Theme pro Level zu setzen — oder live mitten im Gespräch umzuschalten — nutze den `AMayDialogueThemeSetter`-Actor (siehe **Mehrere Themes im selben Projekt** unten); genau so setzt jede Sample-Map ihr Theme.

> 📸 **Bild-Platzhalter:** `themes-project-settings.png` — Project Settings → MayDialogue. Feld `DefaultDialogueWidgetClass` mit einem ausgefüllten Widget-Pfad, roter Pfeil darauf.
> *Setup:* Project Settings → MayDialogue, Screenshot nur diesen Abschnitt.

---

## Horror-Theme

> 📸 **Bild-Platzhalter:** `theme-horror-ingame.png` — PIE-Viewport, Horror-Theme aktiv (Sample-Map `L_Horror_Corridor` anspielen). Fast schwarzes Panel (1120×240) unten zentriert, Speaker-Nameplate links darüber in Special-Elite-Typewriter-Schrift, fahler grauweißer Text. Einblend-Animation fast fertig (CRT-Flicker-Fade).
> *Setup:* `L_Horror_Corridor` in PIE starten, an den NPC herantreten (Proximity-Start). Screenshot kurz nach dem Einblenden.

**Dialog Frame**
- Panel: fast schwarz (`#131316`), 1120×240 @1080p, unten zentriert.
- Einblend-Animation: CRT-Flicker-Fade (~220 ms — die Deckkraft springt mehrfach, bevor sie steht).
- Ausblend-Animation: schneller Fade (~160 ms).

**Speaker**
- Nameplate links über dem Frame; Name in **Special Elite** (Typewriter-Look), 20 pt, gesperrte Laufweite.
- Ein `PortraitImage`-Slot (96×96) existiert im Widget, ist aber standardmäßig collapsed — hänge eine Textur per Subclass an, wenn du Portraits willst.

**Text**
- Body-Font: Roboto 18 pt in fahlem Grauweiß (`#D8D4CC`).
- Typewriter-Effekte über Rich-Text-Tags im Zeilentext: `<shake>Tot.</shake>`, `<speed>`-Variationen — siehe [Rich-Text-Tags](rich-text-tags.md).

**Choice-Buttons**
- 720×56, gestapelt über dem Frame; eigene Normal/Hover/Pressed/Disabled-Texturen.
- Hover: Opacity-Nudge-Animation (`Anim_Hover`, ~80 ms).
- Gesperrte-aber-sichtbare Choices (`FailedButVisible`): **Lock-Icon** wird eingeblendet, Text wechselt auf die Disabled-Farbe (`#8A857E`).

> 📸 **Bild-Platzhalter:** `theme-horror-choice-hover.png` — PIE-Viewport, Horror-Theme, PlayerChoice aktiv. Button 1 gehovered (Hover-Textur + aufgehellter Text). Ein gesperrter Button mit Lock-Icon und abgedunkeltem Text. Maus-Cursor sichtbar.
> *Setup:* Horror-Theme, PlayerChoice-Node mit 3 Choices (1 davon mit nicht erfülltem Requirement, FailResult = FailedButVisible), Maus über Button 1 hovern, Screenshot.

---

## Visual-Novel-Theme

> 📸 **Bild-Platzhalter:** `theme-vn-ingame.png` — PIE-Viewport, VN-Theme aktiv (Sample-Map `L_VN_Scene` anspielen). Tiefviolettes, breites Panel (1400×300) unten zentriert, Speaker-Name als abgerundete Pill in Quicksand, heller Nunito-Sans-Fließtext. Weicher Fade abgeschlossen.
> *Setup:* `L_VN_Scene` in PIE starten, an Jun/Riley herantreten — der Speaker-Wechsel pro Zeile ist Teil des Samples. Screenshot nach abgeschlossener Einblende.

**Dialog Frame**
- Panel: tiefes Violett (`#322B4D`), 1400×300 @1080p — das breiteste der drei Themes.
- Einblend-Animation: weicher Fade (~280 ms), Ausblenden ~220 ms.

**Speaker**
- Name-**Pill** (abgerundet) statt eckigem Nameplate; Name in **Quicksand SemiBold**, 17 pt.
- Großzügiger `PortraitImage`-Slot (384×480) für VN-Portraits — standardmäßig collapsed; per Subclass mit Texturen belegen.

**Text**
- Body-Font: **Nunito Sans** 20 pt in fast weißem Ton (`#F7F4FA`) — das einzige Theme mit eigener Fließtext-Schrift.
- Typewriter mittlere Geschwindigkeit.

**Choice-Buttons**
- **Schweben mittig im Bild** (Center-Anchor) statt über dem Frame — klassisches VN-Entscheidungs-Layout, 840×64 pro Zeile.
- Gesperrte Choices zeigen Lock-Icon + Disabled-Farbe (`#A9A1BE`).

> 📸 **Bild-Platzhalter:** `theme-vn-choice-layout.png` — PIE-Viewport, VN-Theme, drei mittig schwebende Choice-Zeilen (Center-Anchor, nicht am Frame). Button 2 gehovered (Hover-Textur). Panel mit laufender Frage unten sichtbar.
> *Setup:* VN-Theme, PlayerChoice mit 3 Choices, Maus über Button 2, Screenshot des ganzen Viewports (damit das Schweben sichtbar ist).

---

## RPG-Theme

> 📸 **Bild-Platzhalter:** `theme-rpg-ingame.png` — PIE-Viewport, RPG-Theme aktiv (Sample-Map `L_DialogueShowcase` anspielen, z.B. Gatekeeper Marrow). Dunkles Slate-Panel (1260×232) mit Gilt-Rahmen unten zentriert, Cinzel-Nameplate links darüber, elfenbeinfarbener Text. Drei Choice-Buttons über dem Frame, einer gesperrt mit Lock-Icon.
> *Setup:* `L_DialogueShowcase` in PIE starten, an Marrow herantreten — sein Dialog hat von Haus aus eine gesperrt-sichtbare Gate-Choice.

**Dialog Frame**
- Panel: dunkles Slate (`#161A21`) mit vergoldetem 9-Slice-Rahmen, 1260×232 @1080p, unten zentriert.
- Einblend-Animation: Fade ~200 ms, Ausblenden ~150 ms.

**Speaker**
- Nameplate links über dem Frame; Name in **Cinzel** (Antiqua-Kapitälchen-Look), 17 pt, leicht gesperrt.
- `PortraitImage`-Slot (96×96) vorhanden, standardmäßig collapsed.

**Text**
- Body-Font: Roboto 18 pt in Elfenbein (`#EDE4D3`) — der Text trägt die Pergament-Anmutung, nicht das Panel.
- Typewriter schnell.

**Choice-Buttons**
- 560×52, gestapelt über dem Frame, vergoldete Hover-Textur.
- Gesperrte-aber-sichtbare Choices: **Lock-Icon** + Disabled-Farbe (`#978F7F`) — im Marrow-Sample direkt erlebbar.

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

`DefaultDialogueWidgetClass` (oben) ist der projektweite Default, aber du bist **nicht** auf ein Theme pro Projekt beschränkt. Das Subsystem hält ein **welt-bezogenes Widget-Klassen-Override**, mit dem du ein Theme pro Level wählen oder live mitten im Gespräch umschalten kannst.

**Präzedenz (welche UI das Subsystem auto-spawnt):**

```text
Welt-Override  >  bUseSlateDialogueWidget  >  DefaultDialogueWidgetClass  >  eingebauter UMayDialogueWidget-Fallback
```

Das Welt-Override setzt du auf zwei Wegen:

* **`AMayDialogueThemeSetter`** — ein Drop-in-Level-Actor. Setze seine `Theme Widget Class` für ein statisches Theme pro Level (null Code) und/oder fülle seine `EventThemes`-Map, sodass ein `FireEvent`-Tag eines Dialogs das Theme mitten im Gespräch live umschaltet.
* **`UMayDialogueSubsystem::SetDialogueWidgetClassOverride(WidgetClass)`** — aus Blueprint/C++ für programmatisches Umschalten aufrufen (Einstellungsmenü, Optik pro NPC, Cutscene). Liegt ein Dialog auf dem Bildschirm, wechselt die UI sofort (altes Widget abgebaut, neues gespawnt und neu gebunden); **None** löscht das Override beim nächsten Dialog-Start zurück auf den Projekt-Default.

Das Override ist welt-bezogen, sodass das Reisen in ein Level ohne ThemeSetter automatisch den Projekt-Default zurückbringt. Vollständiger Walkthrough: [Theme-Wechsel zur Laufzeit](../recipes/runtime-theme-switch.md).

Ende UI-Kapitel. Weiter: [Audio-System](../audio/README.md).
