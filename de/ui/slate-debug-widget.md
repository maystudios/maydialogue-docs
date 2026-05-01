---
description: Was das Slate-Debug-Widget zeigt, wann du es an- oder abschaltest, und wie du es durch dein UMG-Widget ersetzt.
---

# Slate-Debug-Widget

`SMayDialogueWidget` ist die eingebaute Slate-UI. Sie startet automatisch, wenn kein UMG-Widget konfiguriert ist — ohne jede zusätzliche Einrichtung.

> 📸 **Bild-Platzhalter:** `slate-widget-ingame.png` — PIE-Viewport: das Slate-Widget im laufenden Dialog. Sichtbar: dunkles Panel unten, Speaker-Name farbig hervorgehoben (Sprecher-Farbe aus dem Graph), Dialog-Text mit laufendem Typewriter, zwei Choice-Buttons nummeriert, blinkender "Weiter"-Hinweis unten rechts.
> *Setup:* Project Settings → MayDialogue → `bUseSlateDialogueWidget = true`, `DefaultDialogueWidgetClass = None`. PIE starten, Dialog triggern.

## Was zeigt das Widget?

Das Layout von unten nach oben:

```
[Viewport]
└── [Panel: dunkel, unten ausgerichtet, Slide-In-Animation]
    ├── [Portrait-Bild] | [Speaker-Name mit farbigem Akzent-Balken]
    │                   | [Trennlinie]
    │                   | [Dialog-Text — Typewriter]
    │                   | [Choice-Buttons mit Nummerierung]
    └──────────────────── [Weiter-Indikator: blinkend]
```

Kein Background-Art, keine Custom-Fonts, keine Animations-Extras — bewusst funktional.

## Wann aktivieren?

Das Widget ist **standardmäßig aktiv**, wenn `DefaultDialogueWidgetClass` leer bleibt.

Typische Szenarien:

- **Prototyping** — du schreibst Dialoge und testest Inhalte, bevor das UMG-Design steht.
- **Debug-Builds** — UI-Polish ist nicht priorisiert, Dialoge müssen aber spielbar sein.
- **Automatischer Fallback** — dein UMG-Widget ist nicht gesetzt oder hat einen Fehler.

## Einschalten / Ausschalten

In den Project Settings → MayDialogue:

```ini
bUseSlateDialogueWidget = true    ; Widget aktiv
DefaultDialogueWidgetClass = None ; kein UMG zugewiesen
```

> 📸 **Bild-Platzhalter:** `slate-project-settings.png` — Project Settings → MayDialogue-Abschnitt. Felder `bUseSlateDialogueWidget` (Checkbox, aktiviert) und `DefaultDialogueWidgetClass` (leer) sind mit roten Pfeilen markiert.
> *Setup:* Edit → Project Settings → Plugins → MayDialogue. Nur diesen Abschnitt screenshotten.

## Rich-Text-Support

Das Slate-Widget unterstützt alle vier visuellen Tags vollständig:

| Tag | Unterstützt |
|---|---|
| `<shake>` | Ja |
| `<wave>` | Ja |
| `<color>` | Ja |
| `<b>` | Ja |
| `<pause>` | Ja (Typewriter-Parser konsumiert ihn) |
| `<speed>` | Ja (Typewriter-Parser konsumiert ihn) |

## Auf UMG wechseln

Wenn dein UMG-Widget fertig ist:

1. Baue dein Blueprint-Widget auf Basis von `UMayDialogueWidget` (siehe [UMG-Architektur](umg-architecture.md)).
2. Setze `DefaultDialogueWidgetClass` in den Project Settings auf dein Widget.
3. `bUseSlateDialogueWidget = false` (optional — das UMG-Widget hat automatisch Vorrang).

Das Slate-Widget ist damit aus dem Workflow.

{% hint style="warning" %}
**Level-Travel:** Das Slate-Widget bindet sich nach einem Level-Wechsel nicht automatisch neu. Rufe vor dem Travel `Subsystem->StopAllDialogues()` auf oder entferne das Widget manuell vom Viewport. Im Blueprint steht dafür der Node `Stop All Dialogues` auf dem MayDialogue-Subsystem bereit.
{% endhint %}
