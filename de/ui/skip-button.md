---
description: Der Skip-Button — wann er sichtbar ist, wie er auf verschiedene Eingabegeräte reagiert, und wie du ihn tauschst.
---

# Skip Button

`UMayDialogueWidget_SkipButton` zeigt dem Spieler, wie er weiterkommen kann — durch Typewriter-Skip oder Dialog-Advance. Das Widget ist platform-aware: es zeigt automatisch den richtigen Hinweis für Tastatur, Gamepad oder Touch.

> 📸 **Bild-Platzhalter:** `skip-button-ingame.png` — PIE-Viewport, laufender Dialog. Skip-Button unten rechts sichtbar: Icon (Leertaste-Symbol bei Tastatur), Text "Weiter". Roter Rahmen um den Skip-Bereich.
> *Setup:* PIE starten, Dialog triggern (Tastatur-Input aktiv). Skip-Button unten rechts screenshotten.

## Wann ist der Skip-Button sichtbar?

Der Skip-Button ist während eines aktiven Dialogs immer sichtbar. Das Top-Level-Widget steuert seine Sichtbarkeit — beim Dialog-Start `ShowFrame`, beim Ende `HideFrame`.

Was beim Klick passiert, entscheidet das Top-Level-Widget:

```
Klick auf SkipButton
  → RequestAdvance() im Top-Level-Widget
      → IsTypewriterActive()?
          True  → SkipTypewriter()   ← Text sofort komplett zeigen
          False → AdvanceDialogue()  ← nächste Zeile / Dialog beenden
```

## BindWidget-Slots

| Name im Designer | Typ | Zweck |
|---|---|---|
| `SkipClickButton` | `UButton` | Der klickbare Bereich |
| `SkipLabel` | `UTextBlock` | Platform-Hinweis (automatisch befüllt) |

Beide sind optional. Ohne `SkipClickButton` kann der Spieler trotzdem per Tastatur advancen (das Top-Level-Widget capturt Keys direkt).

{% hint style="warning" %}
`SkipClickButton` muss exakt so heißen — sonst bleibt der Button-Klick stumm.
{% endhint %}

## Platform-Hinweis konfigurieren

In den Blueprint-Defaults deiner SkipButton-Subklasse:

```ini
KeyboardHint = "Space drücken"   ; bei Tastatur/Maus
GamepadHint  = "A drücken"       ; bei Gamepad
```

Ruf `SetInputDevice(bool bIsGamepad)` auf, wenn sich das aktive Eingabegerät ändert. `SkipLabel` wird automatisch aktualisiert.

```text
Event On Input Device Changed (bIsGamepad)
  → SkipButton → SetInputDevice(bIsGamepad)
```

Für den aktuellen Text: `GetSkipHintText()` gibt den aktiven Hinweis zurück.

## Blueprint Event

```cpp
// Feuert wenn RequestSkip() aufgerufen wird. Für eigene Animationen/Sounds.
void OnSkipRequested()
```

```text
Event On Skip Requested
  → Play Sound (UI_Click)
  → Play Animation "ButtonPulse"
```

> 📸 **Bild-Platzhalter:** `skip-button-event-graph.png` — Blueprint-Graph von WBP_MySkipButton. Event "On Skip Requested" → Play Sound + Play Animation. Daneben: Blueprint-Defaults-Panel mit KeyboardHint "Space drücken" und GamepadHint "A drücken" sichtbar.
> *Setup:* WBP_MySkipButton → Event Graph + Blueprint-Defaults screenshotten.

## Typisches Layout

```
WBP_SkipButton
└── Overlay
    ├── Background (abgedunkelte Ecke)
    ├── Icon-Image  (Plattform-abhängig: Space / Ⓐ / Touch-Symbol)
    └── SkipLabel   (TextBlock, Name: "SkipLabel")
```

> 📸 **Bild-Platzhalter:** `skip-button-umg-designer.png` — UMG-Designer von WBP_MySkipButton. Hierarchy: Overlay → Image (Background) + Image (Icon) + TextBlock (Name "SkipLabel"). Viewport-Preview zeigt das fertige Layout.
> *Setup:* WBP_MySkipButton im UMG-Designer öffnen, Hierarchy und Viewport-Preview screenshotten.

## Eigenen Skip-Button bauen

**Schritt 1** — Blueprint-Subklasse: Parent `MayDialogueWidget_SkipButton`, Name z.B. `WBP_MySkipButton`.

**Schritt 2** — UMG-Designer: `Button` (Name: `SkipClickButton`), `TextBlock` (Name: `SkipLabel`), plus eigene Icons und Hintergrund.

**Schritt 3** — `KeyboardHint` und `GamepadHint` in den Blueprint-Defaults setzen.

**Schritt 4** — Optional: `On Skip Requested` für Sounds oder Animations-Feedback implementieren.

**Schritt 5** — In deinem DialogFrame das `SkipWidget`-Child durch `WBP_MySkipButton` ersetzen.
