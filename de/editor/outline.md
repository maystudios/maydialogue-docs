---
description: Alle Nodes als durchsuchbare Liste – filtern, suchen und per Klick direkt anspringen.
---

# Outline

Der Outline-Tab zeigt alle Nodes deines Dialogs als **sortierte Liste** – mit Sprecher-Chips, Text-Previews und Typ-Badges. Klick auf eine Zeile springt direkt zum Node im Graph.

> 📸 **Bild-Platzhalter:** `outline-overview.png` — Outline-Tab mit sieben Einträgen eines vollständigen Dialogs.
> *Setup:* Outline-Tab aktiv. Sieben Zeilen: Entry (kein Chip, Badge „E"), SayLine Guard „Halt! Wer bist du?" (dunkelroter Chip, Badge „Say"), PlayerChoice „Du antwortest:" (blauer Chip, Badge „PC"), SayLine Guard „Dann passiere." (dunkelroter Chip), SayLine Guard „Dann verzieh dich!" (dunkelroter Chip), Exit Completed (kein Chip, Badge „X"), Exit Failed (kein Chip, Badge „X"). Suchfeld oben leer, Filter auf „All".

## Wofür brauchst du den Outline?

Der Graph zeigt Nodes räumlich – du suchst sie nach ihrer Position. Sobald ein Asset mehr als 20 Nodes hat, wird räumliche Suche mühsam. Der Outline beantwortet Fragen wie:

- *„Wer sagt das nochmal?"* → Sprecher-Chip sofort sichtbar
- *„Wie viele PlayerChoices hat dieser Dialog?"* → Filter: Player Choice
- *„Welche SayLine spricht über den Keller?"* → Suchfeld: „Keller"

## Aufbau einer Zeile

| Element | Bedeutung |
| --- | --- |
| **Farb-Chip** | Sprecher-Farbe (grau für Nodes ohne Sprecher) |
| **Primärtext** | Sprecher-DisplayName oder Node-Titel |
| **Sekundärtext** | Dialog-Text-Preview (gekürzt auf 60 Zeichen) |
| **Typ-Badge** | Zweiletter-Kürzel rechts |

### Typ-Badges Referenz

| Badge | Node-Typ |
| --- | --- |
| `E` | Entry |
| `X` | Exit |
| `Say` | SayLine |
| `PC` | PlayerChoice |
| `Br` | Branch |
| `Rn` | RandomLine |
| `Wt` | Wait |
| `Ln` | Link |
| `Sg` | SubGraph |
| `Ev` | Fire Event |
| `Fx` | Apply Effect |
| `SV` | Set Variable |
| `PS` | Play Sound |
| `CF` | Camera Focus |
| `CS` | Camera Shake |
| `PA` | Play Animation |

## Suchen

Suchfeld am oberen Rand des Outline-Tabs. Die Liste filtert **live** während du tippst.

Durchsucht werden:
- Sprecher-DisplayName und Node-Titel (Primärtext)
- Dialog-Text-Preview (Sekundärtext)
- Typ-Badge-Kürzel (z.B. „Say" zeigt alle SayLines)

> 📸 **Bild-Platzhalter:** `outline-search.png` — Outline mit „Keller" im Suchfeld, zwei gefilterte Treffer sichtbar.
> *Setup:* Outline-Tab, Suchfeld „Keller" eingetippt. Nur noch zwei SayLines sichtbar: „Du weißt was im Keller ist." und „Der Keller ist verriegelt." Beide mit dunkelrotem Guard-Chip. Roter Pfeil auf das Suchfeld.

## Filtern nach Node-Typ

Dropdown rechts neben dem Suchfeld wählt einen Typ-Filter:

| Filter | Zeigt |
| --- | --- |
| **All** | Alle Nodes (Standard) |
| **Say Line** | Nur SayLines |
| **Player Choice** | Nur PlayerChoice-Nodes |
| **Branch** | Nur Branch-Nodes |
| **Actions** | Set Variable, Fire Event, Apply Effect, Play Sound, Kamera- und Animation-Nodes |
| **Flow Control** | Wait, Link, SubGraph |
| **Special** | Entry, Exit, Knot |

Filter und Suchfeld lassen sich kombinieren: Filter auf „Say Line" + Suchtext „Wächter" zeigt alle SayLines des Wächters.

> 📸 **Bild-Platzhalter:** `outline-filter-dropdown.png` — Filter-Dropdown offen, „Player Choice" hervorgehoben, darunter zwei PC-Nodes in der Liste sichtbar.
> *Setup:* Outline-Tab, Filter-Dropdown aufgeklappt. „Player Choice" markiert. Im Hintergrund der Liste zwei PlayerChoice-Einträge mit blauen Chips. Roter Pfeil auf das Dropdown.

## Click-to-Jump

Klick auf eine Outline-Zeile:
1. Selektiert den Node im Graph.
2. Zentriert die Graph-Kamera auf den Node.

Damit findest du jeden Node in einem 150-Node-Asset in unter einer Sekunde – ohne manuelles Scrollen und Suchen im Graph.

## Live-Update

Der Outline aktualisiert sich automatisch, wenn du im Graph Nodes hinzufügst, löschst oder umbenennt. Du musst den Tab nicht neu öffnen oder manuell aktualisieren.

{% hint style="info" %}
In seltenen Fällen, in denen ein Graph-Änderungs-Signal nicht sofort ankommt, hat der Outline einen automatischen Fallback-Sync (Hash-Check im Hintergrund). Die Liste bleibt in jedem Fall synchron.
{% endhint %}

## Typische Einsatzfälle

**„Wer sagt das?"**
Text ins Suchfeld eingeben. Erster Treffer zeigt den Sprecher-Chip und -Namen.

**„Wie viele Choices hat dieser Dialog?"**
Filter auf Player Choice → Zeilenzahl ablesen.

**„Alle Kamera-Nodes prüfen"**
Filter auf Actions → Nur Kamera- und Animations-Nodes sichtbar.

**„Zu einem bestimmten Exit springen"**
Filter auf Special → Beide Exits sichtbar, den richtigen anklicken.

> 📸 **Bild-Platzhalter:** `outline-click-to-jump.png` — Outline mit selektierter Zeile, im Hintergrund springt der Graph auf den zugehörigen Node.
> *Setup:* Outline-Tab, zweite SayLine-Zeile angeklickt (blau markiert). Im Graph-Bereich dahinter zentriert die Kamera auf diese SayLine, Node leicht hervorgehoben (Selektion-Farbe). Roter Pfeil auf die selektierte Zeile im Outline.

Weiter: [Find-in-Dialogue →](find.md)
