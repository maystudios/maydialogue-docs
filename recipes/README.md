---
description: Fertige Lösungs-Muster für wiederkehrende Dialog-Aufgaben – nachbaubar in 5–20 Minuten.
---

# Rezepte

Die Rezept-Galerie sammelt **fertige Lösungs-Muster** für wiederkehrende Dialog-Aufgaben. Jedes Rezept ist so aufgebaut, dass du es in ca. 10–20 Minuten nachbauen und auf dein Projekt übertragen kannst.

## Alle Rezepte auf einen Blick

| # | Rezept | Schwierigkeit | Tags |
|---|--------|---------------|------|
| 1 | [Einfaches NPC-Gespräch](simple-npc-talk.md) | Anfänger | Grundlagen |
| 2 | [Zufällige Begrüßungen](random-greetings.md) | Anfänger | Branching |
| 3 | [Verzweigungen mit Bedingungen](branching-conditions.md) | Anfänger | Branching, GAS |
| 4 | [Wiederverwendbare Dialog-Fragmente](linking-dialogues.md) | Anfänger | Struktur |
| 5 | [SubGraph-Organisation](subgraph-organization.md) | Anfänger | Struktur |
| 6 | [Choice nur sichtbar mit Tag](choice-with-tag-requirement.md) | Anfänger | Branching, GAS |
| 7 | [Choice mit Attribut-Bedingung](choice-with-attribute-requirement.md) | Anfänger | Branching, GAS |
| 8 | [GAS-getriebener Dialog](gas-driven-dialogue.md) | Fortgeschritten | GAS |
| 9 | [GameplayEffect aus Dialog anwenden](apply-gameplay-effect.md) | Fortgeschritten | GAS |
| 10 | [Quest-Status im Dialog lesen](quest-status-in-dialogue.md) | Fortgeschritten | GAS, Branching |
| 11 | [Dialog setzt Quest-Progress](dialogue-sets-quest-progress.md) | Fortgeschritten | GAS |
| 12 | [Beziehungs-Score tracken](relationship-counter.md) | Fortgeschritten | Persistence |
| 13 | [NPC erinnert sich beim 2. Treffen](npc-remembers-meeting.md) | Fortgeschritten | Persistence |
| 14 | [Mehrsprachigkeit](multilingual-dialogue.md) | Fortgeschritten | Audio |
| 15 | [Innerer Monolog mit 2D-Audio](inner-monologue-2d.md) | Fortgeschritten | Audio |
| 16 | [Kamera-Schwenk auf Sprecher](camera-pan-on-speaker.md) | Fortgeschritten | Camera |
| 17 | [Jump-Scare mit Camera-Shake](jump-scare-shake.md) | Fortgeschritten | Camera, Animation |
| 18 | [NPC-Animation während Zeile](npc-animation-during-line.md) | Anfänger | Animation |
| 19 | [Timed Choice (Auto-Auswahl)](timed-choice.md) | Anfänger | Branching, UI |
| 20 | [Wait auf externes GameplayEvent](wait-for-event.md) | Fortgeschritten | GAS |
| 21 | [Eigenes UMG-Widget anbinden](custom-umg-widget.md) | Fortgeschritten | UI, Extension |
| 22 | [Eigenen Requirement in Blueprint bauen](custom-blueprint-requirement.md) | Fortgeschritten | Extension |

## Empfohlene Reihenfolge für Einsteiger

Wenn du MayDialogue zum ersten Mal nutzt, empfiehlt sich dieser Pfad:

```text
1. Einfaches NPC-Gespräch
       │
       ├─► Zufällige Begrüßungen
       │
       └─► Verzweigungen mit Bedingungen
                   │
                   ├─► Choice nur sichtbar mit Tag
                   ├─► Choice mit Attribut-Bedingung
                   └─► GAS-getriebener Dialog
                               │
                               ├─► Quest-Status im Dialog lesen
                               ├─► Dialog setzt Quest-Progress
                               └─► GameplayEffect anwenden

Parallel dazu (Struktur):
       Wiederverwendbare Fragmente → SubGraph-Organisation

Fortgeschritten (nach Bedarf):
       Beziehungs-Score → NPC erinnert sich
       Kamera → Jump-Scare → NPC-Animation
       Timed Choice → Wait auf Event
       Eigenes Widget → Eigener Requirement
```

## Voraussetzungen

Vor dem ersten Rezept solltest du:

1. Das [Quick-Start-Tutorial](../getting-started/quick-start.md) absolviert haben.
2. Die drei Konzeptseiten gelesen haben:
   - [Instance & Lifecycle](../concepts/instance-lifecycle.md)
   - [Participants & Sprecher](../concepts/participants-speakers.md)
   - [Variablen & Scopes](../concepts/variables-scopes.md)

{% hint style="info" %}
Alle Rezepte gehen davon aus, dass du das Plugin laut [Installation](../getting-started/installation.md) integriert hast und in den [Projekt-Einstellungen](../getting-started/project-settings.md) ein Dialog-Widget konfiguriert ist.
{% endhint %}

## Konventionen in den Code-Snippets

| Kürzel | Bedeutung |
|--------|-----------|
| `Sub` | `UMayDialogueSubsystem*` |
| `Inst` | `UMayDialogueInstance*` |
| `Asset` | `UMayDialogueAsset*` |
| `PC` | `APlayerController*` |
| `NPC` | Gesprächspartner-Actor |

Vollständige Boilerplate findest du in [Runtime → Einen Dialog starten](../runtime/starting-dialogues.md).

## Weiterführende Themen außerhalb der Rezepte

- **Eigene Nodes schreiben** → [Erweiterung → Custom Nodes](../extension/custom-nodes.md)
- **SaveGame-Integration** → [Persistence](../persistence/README.md)
- **UI-Theming** → [UI → Themes & Starterkits](../ui/themes.md)
- **Audio-Fallback-Kette** → [Audio → Drei-Ebenen-Fallback](../audio/three-level-fallback.md)
