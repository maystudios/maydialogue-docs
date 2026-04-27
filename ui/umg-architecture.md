---
description: Die fünf Widget-Klassen im Überblick, wie sie zusammenspielen, und wie du einzelne Bausteine gegen deine eigenen tauschst.
---

# UMG-Architektur

Das UMG-UI ist **komponenten-basiert**. Jede Widget-Klasse hat eine klar abgegrenzte Aufgabe. Du subclasst genau die Teile, die du anpassen willst — den Rest lässt du unverändert.

> 📸 **Bild-Platzhalter:** `umg-component-diagram.png` — Schaubild der Komponentenbeziehung. Weißer Hintergrund. Oberstes Element: Rechteck "UMayDialogueWidget (Top-Level)" mit gestrichelter Umrandung. Darunter per Pfeil verbunden: Rechteck "DialogFrame". Vom DialogFrame gehen vier Pfeile ab zu: "Speaker" (links), "Text" (mittig-links), "ChoiceList" (mittig-rechts), "SkipButton" (rechts). Von ChoiceList geht ein weiterer Pfeil ab zu "ChoiceButton (n×)". Alle Verbindungen sind beschriftet: "BindWidget". Layout horizontal, minimalistisch.
> *Setup:* Diagramm als Vektorgrafik oder einfaches Blueprint-Widget-Hierarchie-Screenshot aus dem UMG-Designer.

## Die Klassen auf einen Blick

| Klasse | Aufgabe |
|---|---|
| `UMayDialogueWidget` | Top-Level. Bindet sich an Instance/Participant, verteilt Events. |
| `UMayDialogueWidget_DialogFrame` | Container. Hält Sub-Widgets, steuert Open/Close-Animationen. |
| `UMayDialogueWidget_Speaker` | Name + Portrait + Emotion-Tags. |
| `UMayDialogueWidget_Text` | Typewriter-Text mit Rich-Text-Decorators. |
| `UMayDialogueWidget_ChoiceList` | Spawnt und verwaltet ChoiceButtons. |
| `UMayDialogueWidget_ChoiceButton` | Einzelner Antwort-Button. |
| `UMayDialogueWidget_SkipButton` | Weiter-Prompt, Platform-aware. |

Alle Klassen sind `Abstract` und `Blueprintable`. Du arbeitest immer mit Blueprint-Subklassen.

## Daten-Fluss

```
Instance/Participant
    │  OnMessageReceived
    ▼
UMayDialogueWidget (Top-Level)
    ├──► DialogFrame → OnDialogueStarted / OnDialogueEnded
    ├──► Speaker     → SetSpeakerData(Name, Portrait, EmotionTags)
    ├──► Text        → StartTypewriter(Text, CPS)
    │         └──► OnCharacterRevealed (Hook für Babel-Synthese)
    ├──► ChoiceList  → SetChoices(Entries)
    │         └──► OnChoiceSelected → RequestSelectChoice(Index)
    └──► SkipButton  → OnSkip → RequestAdvance()
```

## Zwei Konfigurationspfade

### Pfad A — Ein zusammengesetztes Blueprint-Widget (empfohlen)

Du baust ein einziges Top-Level-Blueprint `WBP_MayDialogue` auf Basis von `UMayDialogueWidget`. Darin nesting du die Sub-Widgets direkt:

```
WBP_MayDialogue (Parent: UMayDialogueWidget)
└── Canvas Panel
    └── WBP_DialogFrame (Name: "DialogFrameWidget")
        ├── WBP_Speaker    (Name: "SpeakerWidget")
        ├── WBP_Text       (Name: "TextWidget")
        ├── WBP_ChoiceList (Name: "ChoiceListWidget")
        └── WBP_SkipButton (Name: "SkipButtonWidget")
```

Die Namen der Child-Widgets müssen exakt mit den `BindWidget`-Property-Namen übereinstimmen. UMG verdrahtet sie automatisch.

Dann in den Project Settings:

```ini
[MayDialogue]
DefaultDialogueWidgetClass = /Game/UI/WBP_MayDialogue.WBP_MayDialogue_C
```

> 📸 **Bild-Platzhalter:** `umg-designer-hierarchy.png` — UMG-Designer, Hierarchy-Panel links. Sichtbar: `WBP_MayDialogue` als Root, darunter eingerückt `WBP_DialogFrame` (Name im Feld "DialogFrameWidget"), darunter `WBP_Speaker` (Name "SpeakerWidget"), `WBP_Text` (Name "TextWidget"), `WBP_ChoiceList` (Name "ChoiceListWidget"), `WBP_SkipButton` (Name "SkipButtonWidget"). Roter Pfeil auf die Namens-Felder.
> *Setup:* UMG-Designer mit WBP_MayDialogue öffnen, Hierarchy-Panel screenshotten.

### Pfad B — Per-Klassen-Defaults in den Settings

Du gibst in den Project Settings für jeden Slot eine Klasse an:

```ini
DefaultSpeakerWidgetClass  = /Game/UI/WBP_MySpeaker.WBP_MySpeaker_C
DefaultTextWidgetClass     = /Game/UI/WBP_MyText.WBP_MyText_C
DefaultChoiceListWidgetClass = ...
```

Das System kombiniert sie zur Laufzeit. Nützlich, wenn du unterschiedliche Widget-Kombinationen pro Szenen-Typ brauchst.

Pfad A ist übersichtlicher für die meisten Projekte. Pfad B ist flexibler bei mehreren Theme-Varianten.

## Wie tausche ich einen einzelnen Baustein?

Am Beispiel Speaker-Widget — das Vorgehen gilt für alle Komponenten:

**Schritt 1 — Blueprint-Subklasse anlegen**

`Content Browser → Rechtsklick → Blueprint Class → Parent: MayDialogueWidget_Speaker`
Nenne sie z.B. `WBP_MySpeaker`.

> 📸 **Bild-Platzhalter:** `umg-new-blueprint-speaker.png` — "Pick Parent Class"-Dialog in UE5. Die Suchleiste zeigt "MayDialogueWidget_Speaker", der Treffer ist markiert. Roter Pfeil auf den Eintrag.
> *Setup:* Content Browser → Rechtsklick → Blueprint Class → Suchfeld mit "MayDialogueWidget_Speaker" befüllen.

**Schritt 2 — Layout bauen**

Im UMG-Designer von `WBP_MySpeaker`:
- Lege einen `Image`-Node an, benenne ihn **`PortraitImage`** (exakter Name für BindWidget).
- Lege einen `Text Block` an, benenne ihn **`NameText`**.
- Füge beliebig weitere Elemente hinzu (Emotion-Icons, Hintergrund-Frame etc.).

**Schritt 3 — On Speaker Changed implementieren**

Im Event-Graph: `On Speaker Changed` überschreiben.

```text
Event On Speaker Changed (DisplayName, Portrait, EmotionTags)
  → Set Text (NameText, DisplayName)
  → Set Brush from Texture (PortraitImage, Portrait)
  → Branch: EmotionTags contains "Dialogue.Emotion.Scared"
      True  → Set Portrait to P_Scared, Play Anim "Shake"
      False → Branch: contains "Dialogue.Emotion.Angry"
                  True  → Set Portrait to P_Angry, Set Text Color Red
                  False → Set Portrait to P_Neutral
```

> 📸 **Bild-Platzhalter:** `umg-speaker-event-graph.png` — Blueprint-Graph von WBP_MySpeaker. Event "On Speaker Changed" (lila Event-Node) verbunden mit "Set Text"-Node (NameText-Target), dann "Set Brush From Texture"-Node (PortraitImage-Target), dann "Branch"-Node (Condition: EmotionTags Contains Tag). True/False-Pins gehen zu weiteren Set-Portrait-Nodes.
> *Setup:* WBP_MySpeaker → Event Graph → On Speaker Changed implementieren wie beschrieben. Graph screenshotten.

**Schritt 4 — Einbauen**

In `WBP_MayDialogue` (oder deinem DialogFrame): das bestehende Speaker-Widget-Child gegen `WBP_MySpeaker` tauschen. Name bleibt `SpeakerWidget`.

{% hint style="success" %}
Das war's. Das System ruft automatisch `SetSpeakerData` auf deiner neuen Klasse auf — kein weiterer Verdrahtungsaufwand.
{% endhint %}

## Typische Fehler

| Problem | Lösung |
|---|---|
| SkipButton reagiert nicht | Child-Widget muss exakt `SkipButtonWidget` heißen. |
| Portrait erscheint nicht | Portrait ist `TSoftObjectPtr` — async load. Warte auf `OnSpeakerChanged`. |
| Typewriter ruft Babel nicht auf | Im Component-Pfad manuell binden: `TextWidget → OnCharacterRevealed → BabelSynth`. |
| Sub-Widget-Slot ist null | Widget-Name im Designer stimmt nicht mit Property-Name überein. |

Weiter: [Dialog Frame](dialog-frame.md).
