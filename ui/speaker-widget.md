# Speaker Widget

`UMayDialogueWidget_Speaker` zeigt den **aktuellen Sprecher** an.

## BindWidget-Slots

| Slot | Typ | Zweck |
| --- | --- | --- |
| `NameText` | `UTextBlock` | Sprecher-DisplayName. |
| `PortraitImage` | `UImage` | Sprecher-Portrait. |

## Methoden

```cpp
UFUNCTION(BlueprintCallable)
void SetSpeakerData(
    const FText& DisplayName,
    UTexture2D* Portrait,
    const FGameplayTagContainer& EmotionTags);

UFUNCTION(BlueprintImplementableEvent)
void OnSpeakerChanged();
```

Wird vom Top-Level-Widget bei jedem Speaker-Wechsel aufgerufen.

## Emotion-Tags als Hook

Der Parameter `EmotionTags` ist **der Schlüssel**, um Portraits oder Styles nach Emotion zu variieren:

```
Event OnSpeakerChanged:
  if EmotionTags contain "Dialogue.Emotion.Scared":
    Set Portrait Image Brush to P_Guard_Scared
    Play Animation "Shake"
  elif EmotionTags contain "Dialogue.Emotion.Angry":
    Set Portrait Image Brush to P_Guard_Angry
    Set Text Color to (1.0, 0.2, 0.2)
  else:
    Set Portrait Image Brush to P_Guard_Neutral
```

## Portrait-Auflösung

Das Portrait kommt aus zwei Quellen:

1. **Speaker im Asset** (`FMayDialogueSpeaker::Portrait`).
2. **Fallback**: Participant-Komponente (`UMayDialogueParticipant::Portrait`), falls der Sprecher keines hat.

Das Top-Level-Widget resolvt `TSoftObjectPtr<UTexture2D>` und reicht das fertige `UTexture2D*` ins Speaker-Widget.

## Typisches Design-Pattern

```
[WBP_Speaker]
├── Horizontal Box
│   ├── PortraitImage (Size: 128x128, circular mask)
│   └── Vertical Box
│       ├── NameText (Font: 24pt Bold, Color: bound to Speaker NodeColor)
│       └── EmotionIconRow (optional)
```

## Anmerkungen

* Portrait-Loads sind **synchron**, sobald das Top-Level-Widget sie angefordert hat – kurze Hitches möglich bei Cold-Loads. Workaround: `AsyncLoad` im Level-Startup.
* `FMayDialogueSpeaker::NodeColor` kannst du in deinem Blueprint als Text-Color nutzen, um den Sprecher-Farb-Code aus dem Graph in die UI zu tragen.
