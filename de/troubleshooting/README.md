---
description: Schnell das richtige Kapitel finden, wenn etwas nicht funktioniert.
---

# Troubleshooting — Wegweiser

Dieser Bereich beantwortet die häufigsten Fragen, wenn ein Dialog nicht das tut, was du erwartest. Lies zuerst die Diagnose-Fragen unten — in den meisten Fällen findest du die Antwort in unter fünf Minuten.

## Was willst du lösen?

| Symptom | Seite |
| --- | --- |
| Dialog startet gar nicht, nichts passiert | [Häufige Probleme → Dialog startet nicht](common-issues.md#dialog-startet-nicht) |
| Kein Widget sichtbar, obwohl der Dialog läuft | [Häufige Probleme → Widget erscheint nicht](common-issues.md#widget-erscheint-nicht) |
| Choices fehlen oder sind nicht klickbar | [Häufige Probleme → Choices fehlen](common-issues.md#choices-fehlen) |
| Kein Audio / keine Stimme | [Häufige Probleme → Audio fehlt](common-issues.md#audio-fehlt) |
| Babel-Stimme ist stumm | [Häufige Probleme → Babel stumm](common-issues.md#babel-stumm) |
| Variable wird nicht gespeichert / Requirement liest falsche Werte | [Häufige Probleme → Variable nicht persistent](common-issues.md#variable-nicht-persistent) |
| Dialog bleibt nach einer Zeile hängen | [Häufige Probleme → Dialog bleibt hängen](common-issues.md#dialog-bleibt-hängen) |
| Compile-Fehler ohne klaren Hinweis | [Häufige Probleme → Compile-Fehler](common-issues.md#compile-fehler) |
| Sub-Graph kehrt nicht zum Aufrufer zurück | [Häufige Probleme → Sub-Graph-Rückkehr fehlt](common-issues.md#sub-graph-rückkehr-fehlt) |
| Log-Fehler oder Crash | [Debug-Tipps](debugging-tips.md) |
| Bekanntes Problem, suche Workaround | [Bekannte Issues](known-issues.md) |

## Empfohlener Debug-Flow

```
Dialog verhält sich falsch
  │
  ├─ Läuft er im Preview-Runner korrekt?
  │     │
  │     ├─ Nein → Strukturfehler im Asset (Nodes, Links, Requirements)
  │     │          → common-issues.md + Validator-Meldungen prüfen
  │     │
  │     └─ Ja → Problem liegt außerhalb des Assets
  │               ├─ Widget nicht sichtbar → Widget-Setup prüfen
  │               ├─ Participant-Fehler → Participant-Komponente + Tags prüfen
  │               └─ GAS-Abhängigkeit → ASC / Tag-State prüfen
  │
  └─ Debugger + Output-Log → debugging-tips.md
```

{% hint style="info" %}
**Schnellster Einstieg:** Öffne das Asset und klicke **Compile**. Viele Probleme sind Compile-Errors, die der Validator sofort benennt.
{% endhint %}

## Struktur dieses Bereichs

- **[Häufige Probleme](common-issues.md)** — Pro Symptom: Ursache + Lösung.
- **[Debug-Tipps](debugging-tips.md)** — Output-Log, Debugger, Preview-Runner, Isolations-Tests.
- **[Bekannte Issues](known-issues.md)** — Offene Einschränkungen der aktuellen Beta + Workarounds.
