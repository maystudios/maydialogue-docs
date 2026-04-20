# Text Widget

`UMayDialogueWidget_Text` ist die Typewriter-Text-Komponente. Sie kapselt die Animation und liefert Hooks für jede enthüllte Zeichen-Position (Babel-Sync, Character-SFX).

## BindWidget-Slots

| Slot | Typ | Zweck |
| --- | --- | --- |
| `DialogueRichText` | `URichTextBlock` | Rendert den Text mit allen aktiven Decorators. |

## Methoden

```cpp
UFUNCTION(BlueprintCallable)
void StartTypewriter(const FText& FullText, float CharactersPerSecond);

UFUNCTION(BlueprintCallable)
void SkipTypewriter();

UFUNCTION(BlueprintPure)
bool IsTypewriterActive() const;

UFUNCTION(BlueprintPure)
float GetTypewriterProgress() const;

UFUNCTION(BlueprintImplementableEvent)
void OnCharacterRevealed(FString Character, int32 Index);

UFUNCTION(BlueprintImplementableEvent)
void OnTypewriterComplete();
```

## Funktionsweise

1. `StartTypewriter(Text, CPS)` parst Inline-Tags über `FMayDialogueTypewriterParser::Parse()`:
   * `<pause=X>` → Pause-Events.
   * `<speed=X>` → Speed-Change-Events.
   * Visuelle Tags (`<shake>`, `<wave>`, `<color>`, `<b>`) **bleiben im Text** – sie werden vom RichTextBlock-Decorator verarbeitet.
2. `NativeTick` akkumuliert Zeit, enthüllt Zeichen für Zeichen.
3. Pro enthülltem Zeichen: `OnCharacterRevealed` Blueprint-Event.
4. Bei Pause-Events: Akkumulator pausiert.
5. Bei Speed-Events: Multiplikator wird aktualisiert.
6. Am Text-Ende: `OnTypewriterComplete`.

## Rich-Text-Decorator-Registrierung

Der `URichTextBlock` im Text-Widget braucht seine **DecoratorClasses** – typisch:

* `UMayDialogueShakeDecorator`
* `UMayDialogueWaveDecorator`
* `UMayDialogueColorDecorator`
* `UMayDialogueBoldDecorator`

Setze sie im Designer-Panel des `DialogueRichText`-Slot-Widgets.

## Babel-Integration

Das Text-Widget feuert `OnCharacterRevealed`. Der natürliche Hook für Babel-Synthese:

```
Event OnCharacterRevealed:
  BabelSynth → OnCharacterRevealed(Character, Index, TotalCharCount)
```

Im Legacy-Pfad (ohne Sub-Widgets) ist das automatisch verdrahtet. Im Component-Pfad musst du es **manuell** in deinem Blueprint binden (Backlog-Gap 2 aus dem UI-Report).

## Skip-Integration

Wenn der Spieler den Skip-Input triggert:

```cpp
Text->SkipTypewriter();  // springt auf das Text-Ende
```

Das Widget wertet `IsTypewriterActive()` vor jedem Advance-Call im Parent aus: wenn noch Typewriter läuft, wird der Skip-Input als „Typewriter skippen" interpretiert, nicht als „Dialog advance".

## Typisches Design-Pattern

```
[WBP_Text]
└── Vertical Box
    ├── DialogueRichText (mit Decorators)
    └── Auto-Scroll-Hint (optional)
```

Styling passiert über den TextStyle des RichTextBlocks (Font, Color, Default-Brush).

## Anmerkungen

* Pro Frame werden normalerweise **mehrere Zeichen** enthüllt – nicht zwingend eins pro Frame. Das `OnCharacterRevealed`-Event feuert für jedes.
* Bei sehr hoher `CharactersPerSecond` kann das Event pro Frame dutzende Male feuern. Babel- oder SFX-Logik sollte damit umgehen können.
