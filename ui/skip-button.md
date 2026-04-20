# Skip Button

`UMayDialogueWidget_SkipButton` zeigt dem Spieler, wie er weiter advancen oder den Typewriter skippen kann.

## BindWidget-Slots

| Slot | Typ |
| --- | --- |
| `SkipClickButton` | `UButton` |

## Methoden

```cpp
UFUNCTION(BlueprintCallable)
void RequestSkip();

UPROPERTY(BlueprintAssignable)
FOnSkip OnSkip;

UFUNCTION(BlueprintImplementableEvent)
void OnSkipRequested();
```

## Verhalten

1. Player klickt den Button (oder drückt Advance-Input, den das Parent-Widget selbst capturet).
2. `RequestSkip()` feuert `OnSkip`.
3. Parent-Widget bindet `OnSkip` → ruft `Advance` auf Instance oder Participant.

## Input-Mapping

Der Skip-Button reagiert standardmäßig auf:

* **Maus-Klick** auf den Button.
* **Space**, **Enter**, **Gamepad-A** (falls im Parent-Widget gemapped).

Das Parent-Widget capturet Key-Events und leitet sie an `RequestSkip()` weiter.

## Typisches Design-Pattern

```
[WBP_SkipButton]
└── Overlay
    ├── Background (dimmed corner)
    ├── Icon (Button-Symbol, z.B. Xbox-A / Space)
    └── Text ("Weiter" / "Continue" / "Skip")
```

Die Buttons/Icons können je nach Platform umgeschaltet werden (Input-Method-Hook).

## Platform-Hinweise

Ein häufiger Wunsch: die Anzeige ändert sich je nach Input-Device:

```
Event OnInputDeviceChanged:
  if Keyboard: show "Space"
  if Gamepad:  show "Ⓐ"
  if Touch:    show "Tippen zum Fortfahren"
```

## Anmerkungen

* Der Skip-Button ist rein UI – er drückt `RequestAdvance` im Parent-Widget, das prüft ob Typewriter-Skip oder Dialog-Advance dran ist.
* Wenn kein SkipButton-Widget im Parent gebunden ist, kann der Spieler trotzdem advancen (Parent capturet Keyboard direkt).
