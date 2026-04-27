---
description: Die mentalen Modelle, die du brauchst, bevor du Nodes baust.
---

# Kern-Konzepte

Dieser Abschnitt legt das Fundament. Wer diese Konzepte kennt, versteht warum das Plugin so funktioniert wie es funktioniert — und spart sich Frust beim Aufbau komplexer Dialoge.

## Die vier tragenden Ideen

1. **Der Graph ist das Dokument.** Ein Dialog-Asset liest sich wie ein Drehbuch: wer spricht, was sagt er, welche Wahl hat der Spieler, was folgt daraus. Nicht eine Property-Liste.
2. **Das laufende Gespräch ist eine eigenständige Instanz.** Das Asset ist die Blaupause; jedes gestartete Gespräch lebt als eigene Instanz mit einem klar definierten Lebenszyklus — Start, Durchlauf, Ende, Cleanup.
3. **Participants sind die Akteure im Level.** Jeder Actor, der in einem Dialog mitspricht oder zuhört, trägt eine Participant-Komponente. Sie ist seine Identität im Gespräch.
4. **Sub-Nodes halten Logik kompakt.** Requirements, Choices und SideEffects sind keine eigenen Graph-Boxen, sondern Pills im Body eines übergeordneten Nodes.

## Kapitel dieser Sektion

| Seite | Frage, die sie beantwortet |
| --- | --- |
| [Architektur im Großen](architecture.md) | Welche Hauptbausteine hat das Plugin und wie hängen sie zusammen? |
| [Graph & visuelle Sprache](graph-visual-language.md) | Wie lese ich den Graph? Farben, Formen, Sub-Nodes. |
| [Instance & Lifecycle](instance-lifecycle.md) | Was passiert von Start bis Exit — und was bei Abort? |
| [Participants & Sprecher](participants-speakers.md) | Participant-Komponente vs. Speaker-Definition im Asset. |
| [Variablen & Scopes](variables-scopes.md) | Dialogue-Scope oder Participant-Scope — wann nehme ich was? |
| [Emotionen & Tags](emotions-tags.md) | Tags an SayLines setzen, UI und Audio reagieren lassen. |

## Einstieg für eilige Leser

Wer so schnell wie möglich einen Dialog zum Laufen bringen will, liest zuerst [Architektur im Großen](architecture.md) und [Instance & Lifecycle](instance-lifecycle.md). Diese beiden Seiten geben das nötige Gesamtbild.

Für alle anderen: Die Reihenfolge der Kapitel ist so gewählt, dass jedes auf dem vorherigen aufbaut. Von oben nach unten lesen funktioniert am besten.
