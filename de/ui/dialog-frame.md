---
description: Der Dialog-Frame als Container-Widget — was er enthalten muss, welche Schnittstelle er bietet, und wie du Animationen einbaust.
---

# Dialog Frame

`UMayDialogueWidget_DialogFrame` ist der Container, der alle anderen Dialog-Widgets umschließt. Er kümmert sich um Layout, Hintergrund und Open/Close-Animationen. Alle Sub-Widgets sind optional — du nimmst genau die, die du brauchst.

> 📸 **Bild-Platzhalter:** `dialog-frame-ingame.png` — PIE-Viewport, laufender Dialog. Das Frame-Widget ist sichtbar: Hintergrund-Panel unten, darin Speaker oben links, Text in der Mitte, Choice-Buttons darunter, Skip-Hint unten rechts. Roter Rahmen um das gesamte Frame-Widget.
> *Setup:* PIE starten, Dialog triggern. Annotationsrahmen um den Frame-Bereich legen.

## Was der Frame enthalten muss

Der Frame hält vier optionale Sub-Widget-Slots via `BindWidget`. Die Namen im UMG-Designer müssen exakt passen:

| Property-Name | Typ | Zweck |
|---|---|---|
| `SpeakerWidget` | `UMayDialogueWidget_Speaker` | Name + Portrait |
| `TextWidget` | `UMayDialogueWidget_Text` | Typewriter-Text |
| `ChoiceListWidget` | `UMayDialogueWidget_ChoiceList` | Antwort-Buttons |
| `SkipWidget` | `UMayDialogueWidget_SkipButton` | Weiter-Prompt |

> 📸 **Bild-Platzhalter:** `dialog-frame-umg-hierarchy.png` — UMG-Designer, Hierarchy-Panel. WBP_DialogFrame als Root. Darunter: Canvas Panel, darin vier Child-Widgets mit ihren exakten Property-Namen sichtbar. Details-Panel rechts zeigt den Namen des markierten Slots.
> *Setup:* WBP_DialogFrame im UMG-Designer öffnen, Hierarchy-Panel screenshotten. Property-Namen mit rotem Pfeil markieren.

## Schnittstelle

### Blueprint Events — hier steuerst du Animationen

```cpp
// Wird aufgerufen wenn ein Dialog startet. Spiele hier deine Einblendeanimation.
void OnDialogueStarted(const FMayDialogueMessage& Message)

// Wird aufgerufen wenn ein Dialog endet. Spiele hier deine Ausblendanimation.
void OnDialogueEnded()
```

### Callable-Funktionen

```cpp
void ShowFrame()   // Default: Visibility = SelfHitTestInvisible
void HideFrame()   // Default: Visibility = Collapsed
```

Überschreibe diese in deiner Blueprint-Subklasse, um Animationen anzuhängen.

## Animationen einbauen — Schritt für Schritt

**Schritt 1 — UMG-Animation erstellen**

Im UMG-Designer → Animations-Tab → `+` → Name: `FrameIntro`. Keyframes für Opacity (0→1) und Position (SlideUp, z.B. Translation Y: 50→0) setzen, Dauer 0,3 s.

> 📸 **Bild-Platzhalter:** `dialog-frame-animation-panel.png` — UMG-Designer, Animations-Tab unten. Animation "FrameIntro" ist markiert, Timeline zeigt zwei Tracks: Opacity (0.0 → 1.0) und Translation (Y: 50 → 0). Dauer 0,3 s.
> *Setup:* UMG-Designer öffnen, Animations-Tab einblenden, FrameIntro-Animation anlegen und screenshotten.

**Schritt 2 — On Dialogue Started implementieren**

```text
Event On Dialogue Started (Message)
  → Set Visibility (Self, SelfHitTestInvisible)
  → Play Animation (FrameIntro, NumLoopsToPlay: 1, PlayMode: Forward)
```

**Schritt 3 — On Dialogue Ended implementieren**

```text
Event On Dialogue Ended
  → Play Animation (FrameOutro, NumLoopsToPlay: 1, PlayMode: Forward)
  → Delay (Outro-Dauer, z.B. 0.25 s)
  → Set Visibility (Self, Collapsed)
```

> 📸 **Bild-Platzhalter:** `dialog-frame-event-graph.png` — Blueprint-Graph des WBP_DialogFrame. Zwei Event-Chains nebeneinander: "On Dialogue Started" → Set Visibility → Play Animation "FrameIntro". "On Dialogue Ended" → Play Animation "FrameOutro" → Delay → Set Visibility Collapsed.
> *Setup:* WBP_DialogFrame Event Graph öffnen, beide Events implementieren, Graph screenshotten.

## Typisches Layout-Muster

```
WBP_DialogFrame
└── Canvas Panel
    ├── Background (Image / Blur)        ← Theme: dunkles Panel, pergament, etc.
    ├── SpeakerWidget  (oben links)
    ├── TextWidget     (Mitte)
    ├── ChoiceListWidget (unten)
    └── SkipWidget     (unten rechts)
```

## Theme-Hooks

Der Frame ist der richtige Ort für alle visuellen Theme-Entscheidungen:

- **Background-Image** — Pergament, Dunkelfläche, oder transparente Box.
- **Border-Style** — Scharfe Ecken für Horror, weiche Rundungen für VN.
- **Color-Scheme** — via Widget-Animations oder Material-Parameter.

Siehe [Themes & Starterkits](themes.md) für fertige Beispiel-Setups.

{% hint style="info" %}
**Eigene Variante:** Lege eine Blueprint-Subklasse von `UMayDialogueWidget_DialogFrame` an. Überschreibe `ShowFrame`/`HideFrame` für eigene Animationslogik. Implementiere `OnDialogueStarted`/`OnDialogueEnded` für Lifecycle-Reaktionen.
{% endhint %}

{% hint style="warning" %}
Der Frame **besitzt** seine Sub-Widgets technisch nicht — er hält Referenzen. Die Sub-Widgets sind Kinder des Top-Level-Widgets. Änderungen an der Sichtbarkeit des Frames betreffen die Sub-Widgets nicht automatisch.
{% endhint %}
