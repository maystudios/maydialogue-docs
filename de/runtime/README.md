---
description: Wie verbinde ich MayDialogue mit meinem Spiel-Code?
---

# Runtime-Integration

Hier lernst du, wie du MayDialogue aus deinem Spiel-Code heraus steuerst: Dialog starten, auf Events reagieren, Variablen lesen und schreiben.

## Drei Wege zum selben Ziel

MayDialogue bietet drei Einstiegspunkte. Alle starten letztlich dieselbe Instance — du wählst den Weg, der am besten in deinen bestehenden Code passt.

| Weg | Klasse | Wann nutzen |
|---|---|---|
| **Participant-Komponente** | `UMayDialogueParticipant` | NPC hat die Komponente am Actor; du willst direkt vom NPC-Blueprint aus starten. Ideal für Interaktions-Trigger. |
| **Blueprint-Library** | `UMayDialogueLibrary` | Schneller One-Liner aus beliebigem Blueprint (Widget, GameMode, LevelScript). Kein Referenz-Caching nötig. |
| **Subsystem direkt** | `UMayDialogueSubsystem` | System-Code (Quest-Director, Cutscene-Regie, Tutorial-Manager). Du brauchst ohnehin das Subsystem für Event-Binding. |

> 📸 **Bild-Platzhalter:** `runtime-three-ways-overview.png` — Vergleichsdiagramm der drei Methoden nebeneinander als drei BP-Graph-Ausschnitte.
> *Setup:* Drei Blueprint-Graphen nebeneinander in einem Screenshot. Links: NPC-Blueprint mit `Get Component by Class (MayDialogueParticipant)` → `Start Default Dialogue`. Mitte: beliebiges Blueprint mit `Start Dialogue` Library-Node (Kategorie MayDialogue). Rechts: `Get MayDialogue Subsystem` → `Start Dialogue` Subsystem-Node.

## Kapitel-Überblick

| Seite | Inhalt |
|---|---|
| [Einen Dialog starten](starting-dialogues.md) | Alle drei Methoden mit Blueprint-Graph und C++-Snippet. |
| [Subsystem-API](subsystem-api.md) | Alle Subsystem-Funktionen mit Use-Case und Beispiel. |
| [Blueprint-Library](library-api.md) | Library-Methoden im Überblick, Blueprint-fokussiert. |
| [Bridge & Lifecycle-Events](bridge-events.md) | Delegates: wann sie feuern, wie du dich einbindest. |
| [Read/Write-API](read-write-api.md) | Variablen lesen/schreiben, Choice auswählen, ForceAdvance. |

## Was im Hintergrund passiert

Du rufst `StartDialogue` auf — der Rest läuft automatisch:

```text
Dein Code  →  Library::StartDialogue(Asset, Instigator, Target)
               ↓
           Subsystem::StartDialogue(...)
               ↓
           Neue Instance, Entry-Node ausführen
               ↓  (Delegates)
           UI-Widget ← OnMessageReceived / OnChoicesPresented
               ↓  (Player-Input)
           Instance::AdvanceDialogue() / SelectChoice()
```

{% hint style="info" %}
Die gesamte Runtime-API ist aus Blueprint erreichbar. C++ brauchst du nur, wenn du eigene Custom-Nodes schreibst oder enge Systemkopplung ohne Blueprint-Overhead willst.
{% endhint %}
