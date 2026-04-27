---
description: ChoiceList und ChoiceButton — wie sie zusammenspielen, welche Layout-Optionen es gibt, und wie du eigene Button-Styles baust.
---

# Choice List & Choice Button

Zwei zusammengehörige Widgets: **ChoiceList** ist der Container, **ChoiceButton** ist der einzelne Antwort-Button. Du ersetzt sie getrennt — eigener Button-Style, ohne die ChoiceList anzufassen.

> 📸 **Bild-Platzhalter:** `choice-list-ingame.png` — PIE-Viewport, PlayerChoice-Node aktiv. Sichtbar: drei Choice-Buttons vertikal angeordnet. Button 1 aktiv (volle Opacity, Hover-Effekt), Button 2 aktiv, Button 3 ausgegraut (unavailable). Roter Rahmen um den gesamten ChoiceList-Bereich.
> *Setup:* Dialog mit einem PlayerChoice-Node (3 Choices, eine davon mit Requirement das nicht erfüllt ist) starten. PIE-Screenshot.

## UMayDialogueWidget_ChoiceList

Der Container spawnt und verwaltet alle ChoiceButtons. Du bindest einen Panel-Container und gibst eine Button-Klasse an.

### BindWidget-Slot

| Name im Designer | Typ | Zweck |
|---|---|---|
| `ChoiceContainer` | `UPanelWidget` (z.B. VerticalBox) | Hier werden Buttons angehängt |

### Property

```cpp
// Welche ChoiceButton-Subklasse soll gespawnt werden?
TSubclassOf<UMayDialogueWidget_ChoiceButton> ChoiceButtonClass;
```

Setze `ChoiceButtonClass` in den Blueprint-Defaults deiner ChoiceList-Subklasse.

### Blueprint Event

```cpp
// Feuert nach SetChoices, wenn alle Buttons gebaut sind.
void OnChoicesSet(int32 ChoiceCount)
```

Hook für z.B. Scroll-Reset, Fokus-Setzen auf den ersten Button, Einblend-Animationen.

### Ablauf intern

```
SetChoices(Entries)
  → ClearChoices()
  → für jeden Entry: spawn ChoiceButton aus ChoiceButtonClass
                     SetChoiceEntry(Entry) aufrufen
                     OnClicked → HandleChoiceClicked(Index)
  → OnChoicesSet(Count) feuern
```

`OnChoiceSelected` (Delegate) → Top-Level-Widget → `RequestSelectChoice(Index)`.

## UMayDialogueWidget_ChoiceButton

Der einzelne Antwort-Button. Jeder Button kennt seinen Index und seinen Availability-Status.

### BindWidget-Slots

| Name im Designer | Typ | Zweck |
|---|---|---|
| `ClickButton` | `UButton` | Der klickbare Button-Bereich |
| `ChoiceTextBlock` | `UTextBlock` | Choice-Text |

### Blueprint Events

```cpp
// Feuert wenn der Button mit Daten befüllt wird. Hier: Styling je nach Availability.
void OnChoiceSet(FText ChoiceText, bool bIsAvailable, FText UnavailableReason)

void OnChoiceHovered()
void OnChoiceUnhovered()
```

### Availability-Status

| Status | Sichtbarkeit | Klickbar |
|---|---|---|
| `Passed` | Sichtbar, volle Opacity | Ja |
| `FailedButVisible` | Sichtbar, ausgegraut | Nein (Tooltip mit Grund) |
| `FailedAndHidden` | Collapsed (kein Layout-Platz) | Nein |

```text
Event On Choice Set (ChoiceText, bIsAvailable, UnavailableReason)
  → Set Text (ChoiceTextBlock, ChoiceText)
  → Branch: bIsAvailable
      True  → Set Color Alpha = 1.0, Set Button Enabled = true
      False → Set Color Alpha = 0.4, Set Button Enabled = false
               Set ToolTip = UnavailableReason
```

> 📸 **Bild-Platzhalter:** `choice-button-event-graph.png` — Blueprint-Graph von WBP_MyChoiceButton. Event "On Choice Set" → Set Text → Branch → True-Pfad (Alpha 1.0, Enabled true) + False-Pfad (Alpha 0.4, Enabled false, Tooltip setzen). Alle Nodes sichtbar.
> *Setup:* WBP_MyChoiceButton → Event Graph → On Choice Set implementieren. Graph screenshotten.

### Konfigurierbare Disabled-Farbe

```cpp
// In Blueprint-Defaults einstellbar:
FLinearColor DisabledTextColor = FLinearColor(0.4, 0.4, 0.4, 1.0)
```

Passe die Farbe pro Blueprint-Subklasse an dein Visual-Theme an.

## Layout-Optionen

**Vertikal (Standard)**

```
WBP_ChoiceList
└── VerticalBox (Name: "ChoiceContainer")
    ├── WBP_ChoiceButton (Choice 0)
    ├── WBP_ChoiceButton (Choice 1)
    └── WBP_ChoiceButton (Choice 2)
```

**Horizontal / Grid**

Tausche `VerticalBox` gegen `HorizontalBox` oder `UniformGridPanel` — der Container ist `UPanelWidget` und damit beliebig ersetzbar.

**Mit Scroll**

Wrapping in einer `ScrollBox`, wenn viele Choices die Sichtfläche überschreiten können:

```
WBP_ChoiceList
└── ScrollBox
    └── VerticalBox (Name: "ChoiceContainer")
```

> 📸 **Bild-Platzhalter:** `choice-list-umg-designer.png` — UMG-Designer von WBP_ChoiceList. Hierarchy: Canvas → VerticalBox (Name "ChoiceContainer"). Details-Panel der VerticalBox sichtbar. Daneben im Viewport: drei Placeholder-Buttons.
> *Setup:* WBP_ChoiceList im UMG-Designer öffnen, Hierarchy und Viewport-Preview screenshotten.

## Eigenen ChoiceButton bauen

**Schritt 1** — Blueprint-Subklasse: Parent `MayDialogueWidget_ChoiceButton`, Name z.B. `WBP_MyChoiceButton`.

**Schritt 2** — UMG-Designer: `Button` (Name: `ClickButton`) und `TextBlock` (Name: `ChoiceTextBlock`) anlegen, plus beliebige eigene Elemente (Index-Nummer, Tasten-Hint, Icon).

**Schritt 3** — `On Choice Set`, `On Choice Hovered`, `On Choice Unhovered` implementieren.

**Schritt 4** — In der Blueprint-Subklasse deiner ChoiceList: `ChoiceButtonClass = WBP_MyChoiceButton` setzen.

{% hint style="success" %}
Du tauschst den Button-Style aus, ohne die ChoiceList anfassen zu müssen. Die ChoiceList spawnt automatisch deine neue Klasse.
{% endhint %}
