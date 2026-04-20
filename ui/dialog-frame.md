# Dialog Frame

`UMayDialogueWidget_DialogFrame` ist der **Container**, der die Dialog-Darstellung auf dem Bildschirm umrahmt. Er kümmert sich um Position, Größe, Hintergrund, Öffnungs-/Schließ-Animation.

## BindWidget-Slots

Optionale Unter-Widget-Slots (alle via `BindWidgetOptional`):

| Slot | Typ |
| --- | --- |
| `SpeakerWidget` | `UMayDialogueWidget_Speaker` |
| `TextWidget` | `UMayDialogueWidget_Text` |
| `ChoiceListWidget` | `UMayDialogueWidget_ChoiceList` |
| `SkipWidget` | `UMayDialogueWidget_SkipButton` |

## Blueprint-Events

```cpp
UFUNCTION(BlueprintImplementableEvent)
void OnDialogueStarted(const FMayDialogueMessage& Message);

UFUNCTION(BlueprintImplementableEvent)
void OnDialogueEnded();
```

Designer implementieren diese Events im Blueprint, um **Open/Close-Animationen** zu triggern (z.B. Einblenden des Hintergrund-Bilds, Fade-In, Slide-Up).

## Methoden

```cpp
UFUNCTION(BlueprintCallable)
void ShowFrame();

UFUNCTION(BlueprintCallable)
void HideFrame();
```

Default-Implementation: einfache Visibility-Änderung. Überschreibbar in Blueprint-Subklassen (*„spiele Widget-Animation 'FadeIn' und dann SetVisibility"*).

## Typisches Design-Pattern

```
[WBP_DialogFrame]
├── Background (Blur/Tint)
├── Top-Bar (Speaker-Slot)
├── Middle (Text-Slot)
├── Bottom (ChoiceList-Slot)
└── Corner (Skip-Slot)
```

Im Blueprint `OnDialogueStarted`:

1. SetVisibility(Visible).
2. Play Animation „FrameIntro" (Slide-Up + Fade-In).

Im Blueprint `OnDialogueEnded`:

1. Play Animation „FrameOutro" (Fade-Out).
2. Nach Animation: SetVisibility(Hidden).

## Theme-Hooks

Typische Theme-Variablen, die das DialogFrame liefert:

* Background-Image (z.B. ein pergamentartiges Banner für RPG).
* Border-Style (scharf-eckig für Horror, weich für VN).
* Color-Scheme (dunkel-rot, hell-gold, pastell-rosa).

Siehe [Themes & Starterkits](themes.md).

## Anmerkungen

* DialogFrame **besitzt** seine Sub-Widgets technisch nicht – es hält nur Referenzen. Die Sub-Widgets sind Kinder des Top-Level-`UMayDialogueWidget`.
* Open/Close-Animationen sind **dein** Job. Das Plugin zwingt kein spezifisches Animation-System auf.
