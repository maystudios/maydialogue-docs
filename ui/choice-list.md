# Choice List & Choice Button

Zwei zusammengehörige Widgets: **ChoiceList** ist der Container, **ChoiceButton** ist der einzelne Antwort-Button.

## UMayDialogueWidget_ChoiceList

### Methoden

```cpp
UFUNCTION(BlueprintCallable)
void SetChoices(const TArray<FMayDialogueChoiceEntry>& Choices);

UPROPERTY(BlueprintAssignable)
FOnChoiceSelected OnChoiceSelected;  // (int32 ChoiceIndex)
```

### Properties

```cpp
UPROPERTY(EditAnywhere)
TSubclassOf<UMayDialogueWidget_ChoiceButton> ChoiceButtonClass;
```

### Ablauf

1. `SetChoices(Entries)` räumt alte Buttons auf.
2. Für jeden `FMayDialogueChoiceEntry`:
   * Spawne einen Button aus `ChoiceButtonClass`.
   * Rufe `SetChoice(Index, Text, bAvailable, UnavailableReason)`.
   * Binde `OnClicked` an `HandleChoiceClicked`.
3. `HandleChoiceClicked` feuert `OnChoiceSelected`.

Das Top-Level-Widget bindet sich an `OnChoiceSelected` und ruft `Instance::SelectChoice(Index)` (oder den RPC-Pfad für Multiplayer).

### Typisches Design-Pattern

```
[WBP_ChoiceList]
└── Vertical Box (Buttons werden hier angehängt)
```

Mit optionalem Auto-Scroll, wenn die Choice-Anzahl die Sichtbare überschreitet.

## UMayDialogueWidget_ChoiceButton {#choicebutton}

### BindWidget-Slots

| Slot | Typ |
| --- | --- |
| `ClickButton` | `UButton` |
| `ChoiceTextBlock` | `UTextBlock` |

### Methoden

```cpp
UFUNCTION(BlueprintCallable)
void SetChoice(int32 Index, const FText& Text, bool bAvailable, const FText& UnavailableReason);

UPROPERTY(BlueprintAssignable)
FOnClicked OnClicked;  // (int32 ChoiceIndex)

UFUNCTION(BlueprintImplementableEvent)
void OnChoiceSet();

UFUNCTION(BlueprintImplementableEvent)
void OnChoiceHovered();

UFUNCTION(BlueprintImplementableEvent)
void OnChoiceUnhovered();
```

### Styling-Flow

`SetChoice` setzt interne Properties und feuert `OnChoiceSet`. Designer implementiert das Event:

```
Event OnChoiceSet:
  if not bAvailable:
    Set Alpha to 0.4
    Set Hoverable to false
    Show Tooltip: UnavailableReason
  else:
    Set Alpha to 1.0
    Set Hoverable to true
```

### Typisches Design-Pattern

```
[WBP_ChoiceButton]
├── ClickButton (Style: custom hover, click-feedback)
│   └── Horizontal Box
│       ├── Index-Number (Font: 18pt, gray)
│       ├── ChoiceTextBlock (Font: 20pt, white)
│       └── Key-Binding-Hint (z.B. "[A]", "[B]", optional)
```

## Anmerkungen

* `SetChoice` wird automatisch vom ChoiceList aufgerufen – du rufst es nie direkt.
* Hover-/Unhovered-Events sind rein UI-lokal; der Dialog-State ändert sich erst bei echtem Click.
* **Gegraute Buttons** (`bAvailable=false`) bleiben sichtbar, aber Click wird ignoriert.
