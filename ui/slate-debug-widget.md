# Slate-Debug-Widget

`SMayDialogueWidget` ist die **Slate-only**-UI-Implementierung. Sie läuft ohne jede UMG-Konfiguration, sobald ein Dialog startet.

## Wann nutzen?

* **Prototyping**. Du schreibst Dialoge, bevor das UMG-Design steht.
* **Debug-Builds**. UI-Polish ist nicht priorisiert, aber Dialoge müssen spielbar sein.
* **Fallback**. Wenn dein UMG-Widget nicht gesetzt oder gebrochen ist.

## Wie aktivieren?

In den [Project Settings](../getting-started/project-settings.md):

* `bUseSlateDialogueWidget = true` (Default).
* `DefaultDialogueWidgetClass = None` (kein UMG zugewiesen).

Sobald ein Dialog startet, taucht das Slate-Widget am Viewport auf.

## Aussehen

Minimalistisch:

* **Sprecher-Name-Leiste** oben (in Sprecher-Farbe).
* **Text-Bereich** mittig mit Typewriter.
* **Choice-Liste** darunter (vertikal angeordnete Buttons).
* **Skip-Hinweis** am unteren Rand.

Kein Portrait, keine Animationen, kein Background-Art – bewusst funktional.

## Rich-Text-Support

Das Slate-Widget nutzt direkt `SRichTextBlock` und registriert Slate-native Decorator-Factories:

* `<shake>`, `<wave>`, `<color>`, `<b>` werden korrekt gerendert.
* `<pause>` und `<speed>` werden vom Typewriter-Parser konsumiert (nicht angezeigt).

## Unterschiede zum UMG-Widget

| | Slate | UMG |
| --- | --- | --- |
| Portraits | Nein | Ja |
| Animationen | Nein | Ja |
| Theming | Hardcoded | Frei |
| Emotion-Tag-Visualisierung | Nein | Optional |
| Blueprint-Erweiterbar | Nein | Ja |

## Abschalten

Sobald dein UMG-Widget fertig ist:

1. `bUseSlateDialogueWidget = false` in Project Settings.
2. `DefaultDialogueWidgetClass` auf dein UMG-Widget setzen.

Das Slate-Widget ist aus dem Workflow.

## Anmerkungen

* Das Slate-Widget wird **nach Level-Travel nicht sauber neu gebunden** (Backlog-Item 1). Workaround: beim Level-Wechsel `Subsystem->StopAllDialogues()` aufrufen oder das Widget manuell aus dem Viewport entfernen.
