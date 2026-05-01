---
description: Das Text-Widget als Typewriter-Komponente — Anbindung, Rich-Text-Decorators, Babel-Hook und Skip-Verhalten.
---

# Text Widget

`UMayDialogueWidget_Text` ist die Typewriter-Komponente. Sie bekommt den Dialog-Text, enthüllt ihn Zeichen für Zeichen, und feuert Events für Babel-Synthese und Skip-Logik.

> 📸 **Bild-Platzhalter:** `text-widget-ingame.png` — PIE-Viewport, Dialog mit laufendem Typewriter. Sichtbar: Partial-Text (ca. 60% enthüllt), Cursor-Blinken am Ende des enthüllten Texts. Rich-Text-Effekte aktiv (ein Wort in Rot, ein Wort mit Shake). Roter Rahmen um den Text-Bereich.
> *Setup:* PIE starten, Dialog mit `<color=red>` und `<shake>`-Tags im Text triggern, Screenshot während Typewriter läuft.

## Anbindung im UMG-Designer

Der einzige BindWidget-Slot:

| Name im Designer | Typ | Zweck |
|---|---|---|
| `DialogueRichText` | `URichTextBlock` | Rendert den Text mit allen aktiven Decorators |

Leg im UMG-Designer einen `RichTextBlock` an und benenne ihn **`DialogueRichText`**. Das Widget befüllt ihn automatisch.

## Typewriter steuern

```cpp
// Startet den Typewriter. CPS <= 0 nutzt den Settings-Default.
void StartTypewriter(FText FullText, float CharactersPerSecond)

// Springt sofort zum vollen Text.
void SkipTypewriter()

// Setzt Text zurück und stoppt den Typewriter.
void ClearText()

// Abfragen
bool IsTypewriterActive()       // läuft der Typewriter noch?
float GetTypewriterProgress()   // 0.0 = kein Text, 1.0 = vollständig enthüllt
```

## Blueprint Events

```cpp
// Feuert für jedes enthüllte Zeichen — Hook für Babel-Synthese und Charakter-SFX.
void OnCharacterRevealed(FString Character, int32 Index)

// Feuert wenn der gesamte Text enthüllt ist (normal oder per Skip).
void OnTypewriterComplete()
```

## Rich-Text-Decorators registrieren

Das `DialogueRichText`-Widget muss seine Decorator-Klassen kennen. Im UMG-Designer:

1. `DialogueRichText` auswählen.
2. Details-Panel → **Decorator Classes** → Elemente hinzufügen:
   - `UMayDialogueShakeDecorator`
   - `UMayDialogueWaveDecorator`
   - `UMayDialogueColorDecorator`
   - `UMayDialogueBoldDecorator`

> 📸 **Bild-Platzhalter:** `text-widget-decorators.png` — UMG-Designer, `DialogueRichText` ausgewählt. Details-Panel rechts zeigt den Abschnitt "Decorator Classes" mit vier Einträgen: Shake, Wave, Color, Bold. Roter Pfeil auf diesen Abschnitt.
> *Setup:* WBP_MyText im UMG-Designer öffnen, DialogueRichText auswählen, Details-Panel screenshotten.

## Babel-Synthese anbinden

Das `OnCharacterRevealed`-Event ist der Hook für Babel-Style-Stimmeffekte:

```text
Event On Character Revealed (Character, Index)
  → BabelSynth → OnCharacterRevealed (Character, Index, TotalCharCount)
```

{% hint style="warning" %}
Im Component-Pfad (Sub-Widget-Architektur) ist die Babel-Anbindung **nicht automatisch**. Du bindest `OnCharacterRevealed` manuell in deinem Blueprint. Referenz auf `BabelSynth` holst du vom Top-Level-Widget.
{% endhint %}

## Skip-Logik

Das Top-Level-Widget prüft vor jedem Advance-Call:

```text
RequestAdvance()
  → IsTypewriterActive()?
      True  → SkipTypewriter()    ← erster Klick: Text komplett zeigen
      False → AdvanceDialogue()   ← zweiter Klick: nächste Zeile / Dialog beenden
```

Aus Spieler-Sicht: erster Klick auf Skip zeigt den vollen Text sofort, zweiter Klick geht weiter.

## Eigenes Text-Widget bauen

**Schritt 1** — Blueprint-Subklasse: Parent `MayDialogueWidget_Text`, Name z.B. `WBP_MyText`.

**Schritt 2** — UMG-Designer: `RichTextBlock` anlegen, Name: `DialogueRichText`. Decorator-Klassen eintragen (s. oben).

> 📸 **Bild-Platzhalter:** `text-widget-umg-designer.png` — UMG-Designer von WBP_MyText. Hierarchy: Canvas → RichTextBlock (Name: "DialogueRichText"). Details-Panel: Font, Farbe, Wrapping eingestellt, Decorator Classes befüllt.
> *Setup:* WBP_MyText im UMG-Designer, Hierarchy + Details-Panel-Abschnitt screenshotten.

**Schritt 3** — Optional: `OnCharacterRevealed` und `OnTypewriterComplete` im Event-Graph für eigene Effekte überschreiben.

**Schritt 4** — In deinem DialogFrame das `TextWidget`-Child durch `WBP_MyText` ersetzen.

{% hint style="info" %}
Styling (Font, Farbe, Zeilenabstand) passiert über den `TextStyle` des RichTextBlocks im UMG-Designer. Der Typewriter-Effekt selbst ist davon unabhängig.
{% endhint %}
