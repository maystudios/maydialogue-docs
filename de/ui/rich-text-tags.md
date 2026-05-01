---
description: Die sechs Core-Tags für Rich-Text im Dialog — pause, speed, shake, wave, color und b — mit Beispielen und Bild-Effekten.
---

# Rich-Text-Tags

MayDialogue unterstützt sechs Inline-Tags direkt im Dialog-Text. Zwei davon steuern den Typewriter-Ablauf, vier erzeugen visuelle Effekte.

## Übersicht

| Tag | Typ | Wirkung |
|---|---|---|
| `<pause=X>` | Typewriter-Control | Pause für X Sekunden |
| `<speed=X>` | Typewriter-Control | Geschwindigkeits-Multiplikator |
| `<shake>...</shake>` | Visuell | Per-Zeichen-Zitter-Effekt |
| `<wave>...</wave>` | Visuell | Sinusförmige Auf-und-Ab-Bewegung |
| `<color=...>...</color>` | Visuell | Farb-Override |
| `<b>...</b>` | Visuell | Fettschrift |

Control-Tags (`pause`, `speed`) werden vom Parser konsumiert und erscheinen nicht im Text. Visuelle Tags bleiben im Text und werden vom `URichTextBlock` gerendert.

---

## `<pause=X>`

Pausiert den Typewriter für X Sekunden. Nützlich für dramatische Pausen.

```text
Er öffnete die Tür.<pause=1.5> Dahinter: nichts.
```

> 📸 **Bild-Platzhalter:** `tag-pause-effect.png` — Zwei PIE-Viewport-Screenshots nebeneinander. Links: Text endet nach "Tür." — Typewriter pausiert, kein weiteres Zeichen. Rechts: nach 1,5 s erscheint " Dahinter: nichts." vollständig. Zeitstempel in der Ecke.
> *Setup:* Dialog mit diesem Beispiel-Text starten, Screenshots direkt vor und nach der Pause machen.

---

## `<speed=X>`

Multipliziert die aktuelle Typewriter-Geschwindigkeit ab dieser Stelle. `<speed=1.0>` setzt auf den Default zurück.

```text
Normal.<speed=0.3> Gaaaaaanz laaangsaaaam...<speed=1.0> Und wieder normal.
```

> 📸 **Bild-Platzhalter:** `tag-speed-effect.png` — PIE-Viewport während des langsamen Abschnitts. Text "Gaaaaaanz laaangsaaaam..." teilweise enthüllt, sichtbar dass der Typewriter sehr langsam läuft. Vergleichs-Screenshot daneben mit normalem Tempo.
> *Setup:* Dialog mit `<speed=0.3>` triggern, Screenshot während des langsamen Abschnitts.

---

## `<shake>...</shake>`

Per-Zeichen-Zitter-Effekt. Jedes Zeichen bewegt sich zufällig um wenige Pixel. Effekt läuft kontinuierlich.

```text
Die Hand<pause=0.5> <shake>zitterte.</shake>
```

> 📸 **Bild-Platzhalter:** `tag-shake-effect.png` — PIE-Viewport, Dialog aktiv. Text "zitterte." ist sichtbar mit per-Zeichen-Shake (Zeichen leicht versetzt, unregelmäßig). Daneben normaler Text zum Vergleich.
> *Setup:* Dialog mit `<shake>zitterte.</shake>` starten, Screenshot im PIE.

### Konfiguration im Designer

Der `UMayDialogueShakeDecorator` hat einstellbare Properties:

```cpp
float ShakeIntensity  = 2.0f;  // Max. Pixel-Offset
float ShakeFrequency  = 15.0f; // Zitter-Geschwindigkeit (Mal/Sekunde)
```

Im UMG-Designer → `DialogueRichText` → Decorator Classes → `UMayDialogueShakeDecorator` auswählen → Details-Panel.

---

## `<wave>...</wave>`

Sinusförmige Auf-und-Ab-Bewegung der Zeichen. Wirkt weich und fließend.

```text
<wave>Das Wasser rauschte.</wave>
```

> 📸 **Bild-Platzhalter:** `tag-wave-effect.png` — PIE-Viewport, Dialog aktiv. Text "Das Wasser rauschte." mit Wave-Effekt: Zeichen auf leicht unterschiedlichen Y-Positionen, Sinus-Form sichtbar. Screenshot muss während der Animation gemacht werden.
> *Setup:* Dialog mit `<wave>Das Wasser rauschte.</wave>` starten, Screenshot im PIE (nicht pausiert).

### Konfiguration im Designer

```cpp
float WaveAmplitude  = 3.0f;   // Max. vertikales Pixel-Offset
float WaveSpeed      = 3.0f;   // Zyklen pro Sekunde
float WaveCharOffset = 0.5f;   // Phasen-Versatz pro Zeichen (in Radiant)
```

---

## `<color=...>...</color>`

Färbt den eingeschlossenen Text in einer anderen Farbe.

```text
Hallo <color=#FF4444>Welt</color>!
Achtung: <color=red>Gefahr!</color>
```

### Unterstützte Formate

| Format | Beispiel | Wirkung |
|---|---|---|
| Hex mit `#` | `<color=#FF0000>` | Rot |
| Hex ohne `#` | `<color=FF0000>` | Ebenfalls Rot |
| Named Color | `<color=red>` | Basis-Farbnamen |

Unterstützte Named Colors: `red`, `green`, `blue`, `yellow`, `white`, `black`, `cyan`, `magenta`, `orange`, `gray`.

> 📸 **Bild-Platzhalter:** `tag-color-effect.png` — PIE-Viewport, Dialog. Text "Hallo " in weißer Default-Farbe, "Welt" in leuchtendem Rot, "!" wieder weiß. Darunter: "Gefahr!" in Rot mit named color. Beide Zeilen sichtbar.
> *Setup:* Dialog mit diesem Beispiel-Text starten, PIE-Screenshot.

---

## `<b>...</b>`

Rendert den Text in Fettschrift. Rein typografisch, keine Animation.

```text
<b>Wichtig:</b> Niemals die rote Tür öffnen.
```

> 📸 **Bild-Platzhalter:** `tag-bold-effect.png` — PIE-Viewport, Dialog. "Wichtig:" deutlich fetter als "Niemals die rote Tür öffnen." daneben. Kontrast zwischen Bold und Normal klar sichtbar.
> *Setup:* Dialog mit diesem Beispiel-Text starten, PIE-Screenshot.

---

## Tags kombinieren

Tags lassen sich verschachteln:

```text
<color=yellow><b>Achtung!</b></color>
<shake><color=red>!!!</color></shake>
Du bist tot. <pause=1.0><color=red><shake>Tot.</shake></color>
```

Ergebnis des letzten Beispiels: "Du bist tot." normal → Pause 1 Sekunde → "Tot." in Rot mit Shake.

> 📸 **Bild-Platzhalter:** `tag-combined-effect.png` — PIE-Viewport, Dialog. Zeile "Du bist tot." normal weiß, dann nach Pause "Tot." in Rot mit aktivem Shake-Effekt. Screenshot muss nach der Pause gemacht werden.
> *Setup:* Dialog mit dem Kombinations-Beispiel starten, Screenshot nach der Pause.

---

## Decorators registrieren

Für UMG-Widgets müssen die vier visuellen Decorators am `URichTextBlock` eingetragen sein:

1. `DialogueRichText` im UMG-Designer auswählen.
2. Details-Panel → **Decorator Classes** → `+`.
3. Eintragen: `UMayDialogueShakeDecorator`, `UMayDialogueWaveDecorator`, `UMayDialogueColorDecorator`, `UMayDialogueBoldDecorator`.

Für das Slate-Debug-Widget sind die Decorators bereits automatisch registriert.

{% hint style="info" %}
**Eigene Decorators:** Erstelle eine Blueprint- oder C++-Subklasse von `URichTextBlockDecorator`. Implementiere `CreateDecorator` und trage sie in die Decorator-Liste deines RichTextBlocks ein. Eigene Tags (z.B. `<flash>`, `<fade>`) sind damit möglich.
{% endhint %}
