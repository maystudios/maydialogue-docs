# Runtime-Integration

Dieser Abschnitt richtet sich an **Gameplay-Programmer**, die MayDialogue in C++ oder Blueprint einbinden. Du lernst:

* Wie du einen Dialog startest und abbrichst.
* Welche APIs `UMayDialogueSubsystem` und `UMayDialogueLibrary` bieten.
* Wie du über die Bridge externen Code an Dialog-Events anschließt.
* Wie du aus externen Systemen Dialog-State liest und schreibst.

## Kapitel-Überblick

* [Einen Dialog starten](starting-dialogues.md) – drei Pfade: Participant, Library, Subsystem.
* [Subsystem-API](subsystem-api.md) – die zentrale Orchestrator-Schnittstelle.
* [Blueprint-Library](library-api.md) – `UMayDialogueLibrary` Blueprint-Callables.
* [Bridge & Lifecycle-Events](bridge-events.md) – `IMayDialogueBridge` plus alle Delegates.
* [Read/Write-API](read-write-api.md) – Variablen, Choices, Advance von außen.

## Kern-Verantwortlichkeiten

```
Code                → Library::StartDialogue(Asset, Instigator, Target)
                     ↓
Library             → Subsystem::StartDialogue(...)
                     ↓
Subsystem           → new Instance, Start, broadcast OnAnyDialogueStarted
                     ↓
Instance            → ContinueToNode(Entry) → ExecuteNode → ...
                     ↓ delegates
UI Widget           → OnMessageReceived / OnChoicesPresented → render
                     ↓ input
UI Widget           → Instance::AdvanceDialogue() / SelectChoice()
```

## Blueprint-First Philosophie

Die komplette Runtime-API ist aus Blueprint erreichbar. C++ ist nur nötig, wenn du **eigene Nodes** oder **enge GAS-Integration** ohne Blueprint-Overhead brauchst.
