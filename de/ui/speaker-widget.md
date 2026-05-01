---
description: Das Speaker-Widget — welche Daten es bekommt, wie du Portrait und Emotion-Tags anbindest, und wie du es komplett ersetzt.
---

# Speaker Widget

`UMayDialogueWidget_Speaker` zeigt Name, Portrait und Emotion des aktuellen Sprechers. Es bekommt alle Daten über einen einzigen Aufruf — `SetSpeakerData` — und feuert dann das Blueprint-Event `On Speaker Changed`, in dem du das Aussehen steuerst.

> 📸 **Bild-Platzhalter:** `speaker-widget-ingame.png` — PIE-Viewport, Dialog aktiv. Das Speaker-Widget ist sichtbar mit Portrait links (kreisförmig beschnitten, 128×128), Sprecher-Name rechts daneben fett, Farbe des Namens entspricht der Sprecher-Farbe aus dem Graphen. Roter Rahmen um den Speaker-Bereich.
> *Setup:* PIE starten, Dialog mit einem Sprecher triggern, der Portrait und NodeColor gesetzt hat.

## Welche Daten bekommt das Widget?

Das Top-Level-Widget ruft bei jedem Sprecher-Wechsel auf:

```cpp
void SetSpeakerData(FText DisplayName, UTexture2D* Portrait, FGameplayTagContainer EmotionTags)
```

Direkt danach feuert das Blueprint-Event `On Speaker Changed` mit denselben Parametern.

### Automatisch verdrahtete Slots

Wenn du diese Widgets im UMG-Designer mit exakten Namen anlegst, werden sie automatisch befüllt:

| Name im Designer | Typ | Befüllt mit |
|---|---|---|
| `NameText` | `UTextBlock` | `DisplayName` |
| `PortraitImage` | `UImage` | Portrait-Texture |

Beide Slots sind optional. Ohne sie musst du alles im `On Speaker Changed`-Event selbst setzen.

## Emotion-Tags nutzen

`EmotionTags` ist ein `FGameplayTagContainer` — du prüfst ihn im Event und reagierst mit Portrait-Wechsel, Animationen oder Farbänderungen.

```text
Event On Speaker Changed (DisplayName, Portrait, EmotionTags)
  → Branch: EmotionTags Contains Tag "Dialogue.Emotion.Scared"
      True  → Set Portrait Brush = P_Speaker_Scared
               Play Animation "Shake"
      False → Branch: Contains "Dialogue.Emotion.Angry"
                  True  → Set Portrait Brush = P_Speaker_Angry
                           Set NameText Color = (1.0, 0.2, 0.2, 1.0)
                  False → Set Portrait Brush = P_Speaker_Neutral
                           Set NameText Color = White
```

> 📸 **Bild-Platzhalter:** `speaker-emotion-graph.png` — Blueprint-Graph. Event "On Speaker Changed" → Branch-Chain für EmotionTags. Jeder Branch-Pfad setzt ein anderes Portrait-Brush und eine andere Farbe. Alle Nodes sichtbar, Pin-Verbindungen deutlich.
> *Setup:* WBP_MySpeaker → Event Graph → On Speaker Changed. Branch-Chain implementieren, screenshotten.

## Portrait-Herkunft

Das Portrait kommt aus zwei Quellen, die das Top-Level-Widget automatisch auflöst:

1. `FMayDialogueSpeaker::Portrait` — Portrait am Sprecher-Asset im Editor.
2. Fallback: `UMayDialogueParticipant::Portrait` am Participant-Component des Actors.

Das Widget bekommt immer ein fertiges `UTexture2D*` — du musst kein async-Loading selbst managen.

{% hint style="warning" %}
Portraits werden bei Cold-Loads synchron geladen — kurze Hitches möglich. Workaround: Portraits via `AsyncLoad` beim Level-Start vorladen.
{% endhint %}

## Sprecher-Farbe aus dem Graph nutzen

Jeder Sprecher im MayDialogue-Editor hat eine `NodeColor`. Diese Farbe steht dir im Widget als `FMayDialogueSpeaker::NodeColor` zur Verfügung. Nutze sie, um den Sprecher-Namen oder die Akzentleiste automatisch in der Graphen-Farbe einzufärben — konsistentes Farb-Coding zwischen Editor und Spiel.

## Eigenes Speaker-Widget bauen

**Schritt 1** — Blueprint-Subklasse anlegen: Parent `MayDialogueWidget_Speaker`, Name z.B. `WBP_MySpeaker`.

**Schritt 2** — UMG-Designer: Image `PortraitImage`, TextBlock `NameText`, plus beliebige eigene Elemente (Emotion-Icon-Row, Hintergrund-Frame, Name-Badge).

> 📸 **Bild-Platzhalter:** `speaker-umg-designer.png` — UMG-Designer von WBP_MySpeaker. Hierarchy-Panel: Canvas Panel → Horizontal Box → Image (Name: "PortraitImage") + Vertical Box → TextBlock (Name: "NameText") + Emotion-Icon-Row. Designer-Vorschau zeigt Layout.
> *Setup:* WBP_MySpeaker im UMG-Designer öffnen, Hierarchy und Viewport-Preview screenshotten.

**Schritt 3** — `On Speaker Changed` im Event-Graph implementieren (siehe oben).

**Schritt 4** — In `WBP_DialogFrame` das alte Speaker-Child-Widget durch `WBP_MySpeaker` ersetzen. Name des Slots bleibt `SpeakerWidget`.

{% hint style="success" %}
Das System ruft `SetSpeakerData` automatisch auf deiner Klasse auf. Kein weiterer Verdrahtungsaufwand.
{% endhint %}
