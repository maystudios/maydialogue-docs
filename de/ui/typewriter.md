---
description: Der Typewriter-Effekt — wie du Geschwindigkeit konfigurierst, Control-Tags einsetzt und Skip-to-End auslöst.
---

# Typewriter-Engine

Der Typewriter-Effekt enthüllt Dialog-Text Zeichen für Zeichen. Er unterstützt Inline-Tags für Pausen und Geschwindigkeitswechsel, und lässt sich jederzeit per Skip auf den vollen Text springen.

> 📸 **Bild-Platzhalter:** `typewriter-ingame-midreveal.png` — PIE-Viewport, Dialog aktiv, Typewriter läuft. Text ca. zur Hälfte enthüllt, sichtbarer Cursor am Ende des enthüllten Teils. Visueller Rich-Text-Effekt (z.B. rotes Wort) bereits sichtbar obwohl Zeichen danach noch nicht enthüllt sind.
> *Setup:* PIE starten, langsamen Dialog triggern (CPS ca. 10), Screenshot genau während des Typewriters machen.

## Grundkonfiguration

In den Project Settings → MayDialogue:

```ini
bEnableTypewriterEffect   = true   ; Typewriter an/aus. false = Text sofort komplett.
TypewriterCharsPerSecond  = 30.0   ; Zeichen pro Sekunde (globaler Default)
```

Wenn `bEnableTypewriterEffect = false` erscheint der gesamte Text sofort — nützlich für Accessibility-Optionen.

> 📸 **Bild-Platzhalter:** `typewriter-project-settings.png` — Project Settings → MayDialogue. Felder `bEnableTypewriterEffect` (Checkbox) und `TypewriterCharsPerSecond` (Zahl 30.0) mit roten Pfeilen markiert.
> *Setup:* Edit → Project Settings → MayDialogue, Screenshot nur diesen Abschnitt.

## Geschwindigkeit pro Dialog überschreiben

Du kannst die Geschwindigkeit pro Dialog-Zeile überschreiben — entweder über den SayLine-Node im Editor oder direkt beim Aufruf:

```cpp
// Im Text-Widget — CPS <= 0 nutzt den Settings-Default
TextWidget->StartTypewriter(Text, CharactersPerSecond);
```

## Control-Tags: Pausen und Geschwindigkeit im Text

Zwei Tags steuern den Typewriter-Ablauf direkt im Dialog-Text. **Sie erscheinen nicht im angezeigten Text** — der Parser konsumiert sie vor der Darstellung.

| Tag | Wirkung |
|---|---|
| `<pause=X>` | Pausiert den Typewriter für X Sekunden |
| `<speed=X>` | Multipliziert die Geschwindigkeit ab dieser Stelle. `<speed=1.0>` setzt zurück |

### Beispiele

```text
Er flüsterte: "Sie... sie ist tot."<pause=1.5> Stille.
```
→ Nach "tot." pausiert der Typewriter 1,5 Sekunden, bevor " Stille." erscheint.

```text
Schnell: <speed=3.0>Er rannte. Sie rannte. Alle rannten.<speed=1.0> Und dann: Stille.
```
→ Der mittlere Teil läuft dreimal so schnell, danach normale Geschwindigkeit.

```text
Du bist tot. <pause=1.0><color=red><shake>Tot.</shake></color>
```
→ Pause, dann roter, zitternder Text.

> 📸 **Bild-Platzhalter:** `typewriter-pause-example.png` — Zwei PIE-Screenshots nebeneinander: links Typewriter direkt nach "tot." (Pause aktiv, kein neues Zeichen enthüllt), rechts nach der Pause mit " Stille." sichtbar. Zeitstempel unten rechts.
> *Setup:* Dialog mit `<pause=1.5>` triggern, Screenshots vor und nach der Pause machen.

## Skip-to-End

Wenn `SkipTypewriter()` aufgerufen wird (z.B. durch Klick auf Skip-Button):

- Der Text springt sofort auf die vollständige Fassung.
- Alle verbleibenden `OnCharacterRevealed`-Events werden **nicht** nachgefeuert.
- `OnTypewriterComplete` feuert einmal.

Das Top-Level-Widget interpretiert den ersten Advance-Klick als Typewriter-Skip, den zweiten als Dialog-Advance:

```text
RequestAdvance()
  → IsTypewriterActive() = true  → SkipTypewriter()
  → IsTypewriterActive() = false → AdvanceDialogue()
```

## Verhältnis zu Rich-Text-Tags

Control-Tags und visuelle Tags können frei gemischt werden:

| Tag | Verarbeitet von | Sichtbar im Text? |
|---|---|---|
| `<pause=X>` | Typewriter-Parser | Nein |
| `<speed=X>` | Typewriter-Parser | Nein |
| `<shake>...</shake>` | RichText-Decorator | Ja |
| `<wave>...</wave>` | RichText-Decorator | Ja |
| `<color=...>...</color>` | RichText-Decorator | Ja |
| `<b>...</b>` | RichText-Decorator | Ja |

Visuelle Tags sind bereits ab dem ersten enthüllten Zeichen aktiv — der Shake-Effekt beginnt, sobald das erste Zeichen des `<shake>`-Abschnitts erscheint.

## Debug-Tipps

| Problem | Lösung |
|---|---|
| Text läuft zu schnell | `TypewriterCharsPerSecond` in Project Settings prüfen |
| Tag erscheint als Text | Leerzeichen im Tag? → `<pause= 0.5>` ist falsch, richtig: `<pause=0.5>` |
| Pause wirkt nicht | Char-Index zählt im geparsten Text (ohne Control-Tags). Das ist korrekt — prüfe Schreibweise |
| `<speed>` setzt sich nicht zurück | `<speed=1.0>` explizit am Ende des Abschnitts setzen |

Siehe [Rich-Text-Tags](rich-text-tags.md) für alle visuellen Tag-Optionen. Für die Babel-Anbindung an `OnCharacterRevealed`: [Audio → Babel-System](../audio/babel-system.md).
