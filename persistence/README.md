---
description: SaveGame-Integration auf einen Blick.
---

# Persistenz

Manche Dialog-Daten müssen über eine Session hinaus leben: ob der Spieler einen NPC schon getroffen hat, wie oft er ihn angesprochen hat, welche Gesprächs-Variante er gewählt hat. MayDialogue speichert das in der `PersistentMemory` auf der Participant-Komponente.

## Philosophie

MayDialogue liefert kein eigenes SaveGame-System. Stattdessen:

* `UMayDialogueParticipant::PersistentMemory` ist mit `UPROPERTY(SaveGame)` markiert — das reicht, damit UE's Standard-Serialisierung es mitnimmt.
* Hast du kein eigenes SaveGame-System, gibt es den eingebauten [QuickSave-Helper](quicksave-helper.md) als Startpunkt.

## Was wird gespeichert — und was nicht

| Gespeichert | Nicht gespeichert |
| --- | --- |
| `PersistentMemory` aller Participants | Aktuell laufende Dialog-Instance |
| Globales `GlobalMemory` (über `UMayDialogueSaveGame`) | Dialogue-Scope-Variablen der laufenden Instance |
| | UI-State |

{% hint style="info" %}
**Dialogue-Scope ist per Definition temporär.** Variablen, die du im Dialog selbst anlegst (Scope = Dialogue), leben nur während des Gesprächs. Wenn du etwas dauerhaft merken willst, schreibe es in `PersistentMemory` (Scope = Participant).
{% endhint %}

## Übersicht: Wie hängt alles zusammen?

```text
Participant A  ──PersistentMemory──┐
Participant B  ──PersistentMemory──┼──► UMayDialogueSaveGame ──► SaveGame-Slot
Participant C  ──PersistentMemory──┘        + GlobalMemory
```

> 📸 **Bild-Platzhalter:** `persistence-overview-diagram.png` — Flussdiagramm: drei Participant-Komponenten fließen in UMayDialogueSaveGame, das in einen Slot gespeichert wird.
> *Setup:* Einfache Blockdiagramm-Grafik (kein UE-Editor nötig). Drei Boxen links ("Participant Guard", "Participant Merchant", "Participant Player") mit Pfeilen nach rechts zu "UMayDialogueSaveGame" (Mitte), von dort ein Pfeil nach rechts zu "SaveGame-Slot AutoSave". Unter UMayDialogueSaveGame zusätzlich "GlobalMemory"-Label.

## Kapitel

* [SaveGame-Integration](save-integration.md) — Einbinden in dein eigenes SaveGame-System.
* [Participant-Memory](participant-memory.md) — Was PersistentMemory kann, Getter/Setter, Events.
* [QuickSave-Helper](quicksave-helper.md) — Der eingebaute Helfer für Projekte ohne eigenes SaveGame.

> 📸 **Bild-Platzhalter:** `persistence-participant-component.png` — Participant-Komponente im Details-Panel eines NPC-Actors.
> *Setup:* Im Level-Editor einen NPC-Actor auswählen. Details-Panel zeigt die `UMayDialogueParticipant`-Komponente. Aufgeklappt sichtbar: `ParticipantTag`, `DisplayName`, `DefaultDialogue`, darunter `PersistentMemory` (ausgegraut / leer, da noch keine Werte gesetzt). Kategorie-Label "Persistence" sichtbar.

> 📸 **Bild-Platzhalter:** `persistence-scope-comparison.png` — Zwei Variablen-Panels nebeneinander: Dialogue-Scope vs. Participant-Scope.
> *Setup:* MayDialogue-Editor offen. Variables-Panel zeigt zwei Gruppen: "Dialogue Scope" (Variablen `DialogueChoice`, `Temp`) und "Participant Scope" (Variablen `HasMet`, `MeetingCount`). Participant-Scope-Variablen sind mit einem Speichern-Symbol markiert.
