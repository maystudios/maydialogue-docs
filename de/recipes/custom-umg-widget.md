---
description: Eigenes UMG-Widget als Dialog-UI anbinden – das Slate-Debug-Widget durch ein produktionsreifes Design ersetzen.
---

# Eigenes UMG-Widget anbinden

## Szenario

Das mitgelieferte Slate-Debug-Widget reicht für Prototypen, aber dein Horrorspiel braucht ein maßgeschneidertes Dialog-UI: verdreckte Schriftart, CRT-Scanlines, handgezeichneter Rahmen. Dieses Rezept zeigt dir, wie du ein eigenes UMG-Widget als Dialog-Frame registrierst und die Spieler-Daten bindest.

## Was du lernst

- Eigenes UMG-Widget als Subklasse des MayDialogue-Frame-Widgets anlegen.
- Widget in den Projekt-Einstellungen registrieren.
- Pflicht-Bindungen: SpeakerName, DialogueText, ChoiceList.
- Typewriter-Engine an den eigenen TextBlock binden.
- Countdown-Binding für Timed-Choice.

## Voraussetzungen

- [Quick Start](../getting-started/quick-start.md) abgeschlossen.
- Grundlegende UMG-Kenntnisse (UserWidget, TextBlock, ListView).

## Was du erstellen wirst

Ein Widget `WBP_DialogFrame_Horror` mit:
- TextBlock für Speaker-Name.
- RichTextBlock für Dialog-Text (Typewriter-fähig).
- ListView oder VerticalBox für die Choice-Buttons.
- ProgressBar für Countdown-Timer.

> 📸 **Bild-Platzhalter:** `custom-umg-widget-final-design.png` — Fertiges Horror-Dialog-Widget im UMG-Editor.
> *Setup:* Widget `WBP_DialogFrame_Horror` im UMG-Editor geöffnet. Sichtbar: dunkler Hintergrund mit Vignette, oben linke Ecke Speaker-Name (weiß, Pixel-Font), mittig großer RichTextBlock, unten Choice-Buttons (3 Stück, VHS-Stil). ProgressBar oben rechts für Countdown.

## Schritt-für-Schritt

### 1. Widget-Basisklasse anlegen

Neues Widget-Blueprint: Parent = **`UMayDialogueFrameWidget`** (MayDialogue-Basisklasse).

Name: `WBP_DialogFrame_Horror`.

{% hint style="info" %}
**Eigene Variante bauen:** `UMayDialogueFrameWidget` ist die Basisklasse für das Dialog-Frame-Widget. Sie deklariert die Pflicht-Interface-Funktionen, die du in der Subklasse überschreiben musst.
{% endhint %}

### 2. UMG-Layout bauen

Im Designer-Panel die folgenden Elemente anlegen:

| Element | Name | Zweck |
|---------|------|-------|
| TextBlock | `SpeakerNameText` | Sprecher-Anzeigename |
| RichTextBlock | `DialogueBodyText` | Zeilen-Text mit Typewriter |
| VerticalBox oder ListView | `ChoiceContainer` | Choice-Buttons |
| ProgressBar | `TimeoutBar` | Countdown-Fortschritt |
| Button-Subwidget | `WBP_ChoiceButton` | Je eine Choice |

> 📸 **Bild-Platzhalter:** `custom-umg-widget-hierarchy.png` — Widget-Hierarchie im UMG-Designer.
> *Setup:* UMG-Designer, linke Panel zeigt Hierarchy: Canvas → Overlay → VBox → [SpeakerNameText, DialogueBodyText, ChoiceContainer (VerticalBox), TimeoutBar]. Alle Elemente mit korrekten Namen in der Liste.

### 3. Pflicht-Funktionen überschreiben

Im Event-Graph des Widgets die folgenden Funktionen implementieren (Override):

**`OnSpeakerChanged(SpeakerData)`:**
```text
[SpeakerData → DisplayName] → [SpeakerNameText → SetText]
```

**`OnLineChanged(LineData)`:**
```text
[LineData → DialogueText] → [MayTypewriterComponent → StartTypewriter(DialogueBodyText)]
```

**`OnChoicesUpdated(Choices)`:**
```text
[Clear ChoiceContainer]
[ForEach Choice: Create WBP_ChoiceButton → Add to ChoiceContainer]
```

**`OnCountdownTick(Progress)`** (für Timed Choice):
```text
[Progress] → [TimeoutBar → SetPercent]
```

> 📸 **Bild-Platzhalter:** `custom-umg-widget-event-graph.png` — Event-Graph mit OnLineChanged-Implementierung.
> *Setup:* WBP_DialogFrame_Horror Event-Graph. Override-Funktion `OnLineChanged` aufgeklappt. Nodes: `LineData → Break FMayDialogueLineData` → `DialogueText` Pin → `MayTypewriterComponent → StartTypewriter` → `DialogueBodyText`. Typewriter-Komponente oben links im Components-Panel.

### 4. Choice-Button-Widget anlegen

Neues Widget `WBP_ChoiceButton`. Enthält:
- Button mit TextBlock für den Choice-Text.
- Optionales Lock-Icon für Requirements.
- `OnClicked` → `MayDialogueLibrary → Submit Choice`.

```text
[Button OnClicked]
   │
   ▼
[MayDialogueLibrary → Submit Choice]
   ├─ ChoiceIndex: (Variable, gesetzt beim Erstellen des Buttons)
   └─ Context:     (aus Parent-Widget)
```

### 5. Widget in Projekt-Einstellungen registrieren

**Edit → Project Settings → MayDialogue → UI**:
- `DialogFrameWidgetClass` = `WBP_DialogFrame_Horror`.

Ab jetzt startet MayDialogue dieses Widget statt des Slate-Debug-Widgets.

### 6. Typewriter binden

Im Widget: **Add Component → MayTypewriterComponent**. In `OnLineChanged`:
```text
[MayTypewriterComponent → StartTypewriter]
   ├─ TargetRichTextBlock: DialogueBodyText
   ├─ Text:                LineData.DialogueText
   └─ CharsPerSec:         60
```

Der Typewriter übernimmt das zeichenweise Aufdecken. `OnTypewriterFinished`-Delegate für AdvanceMode-Logik binden.

> 📸 **Bild-Platzhalter:** `custom-umg-widget-ingame.png` — Horror-Dialog-Widget in PIE mit echtem Content.
> *Setup:* PIE-Screenshot. Widget `WBP_DialogFrame_Horror` sichtbar über dem Spieler. Speaker-Name `"Wächter"` in Pixel-Font. Typewriter läuft durch den Text. Zwei Choice-Buttons unten.

## Variation / Weiter gehen

- **Slate-Debug-Widget** parallel behalten für Quick-Tests: in den Editor-Einstellungen konfigurierbar.
- **Portrait-Bild**: `SpeakerData.Portrait` → `Image → SetBrushFromTexture`.
- **Rich-Text-Tags** für farbige Keywords im Dialog → [UI → Rich-Text-Tags](../ui/rich-text-tags.md).

## Troubleshooting

**Widget erscheint nicht.**
`DialogFrameWidgetClass` in den Projekt-Einstellungen nicht gesetzt oder falscher Widget-Typ (muss von `UMayDialogueFrameWidget` erben).

**Typewriter zeigt Text sofort komplett.**
Typewriter-Komponente nicht gestartet. `StartTypewriter` in `OnLineChanged` prüfen. Oder `bSkipTypewriter = true` in den Projekt-Einstellungen aktiviert.

**Choices werden nicht angezeigt.**
`OnChoicesUpdated` nicht korrekt implementiert oder Choice-Buttons werden dem falschen Container hinzugefügt. Container-Name im Hierarchy-Panel prüfen.
