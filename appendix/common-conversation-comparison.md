# Vergleich mit CommonConversation

Ein tiefes Begleit-Dokument zum Konzept-Kapitel [CommonConversation](../concepts/common-conversation.md). Hier die feature-Seite nebeneinander.

## Zusammenfassung

| Feature | CommonConversation (CC) | MayDialogue |
| --- | --- | --- |
| Runtime-Core | ✅ | ✅ (inspiriert von CC) |
| Plugin-Dependency | (ist selbst Plugin) | **Keine** (eigenständiges Plugin) |
| Multi-Entry-Points pro Asset | ✅ | ❌ (bewusste Entscheidung) |
| Custom-EdGraph-Editor | ❌ (nutzt AIGraph) | ✅ |
| Inline-Node-Text-Edit | ❌ | ✅ |
| Sprecher-Farben in Graph | ❌ | ✅ |
| Sub-Node-Pills im Body | ❌ | ✅ |
| Action-Nodes UND SideEffect-Sub-Nodes | ❌ (nur SideEffect) | ✅ (beide) |
| Built-in UI (Slate + UMG) | ❌ | ✅ |
| Built-in Audio-Pipeline | ❌ | ✅ |
| Babel-Synthese | ❌ | ✅ |
| Typewriter-Engine | ❌ | ✅ |
| Rich-Text-Decorators | ❌ | ✅ |
| Kamera-Nodes | ❌ | ✅ |
| In-Editor-Preview ohne PIE | ❌ | ✅ |
| PIE-Debugger | ✅ | ✅ |
| Outline-Panel | ❌ | ✅ |
| GAS-Integration | ❌ | ✅ (First-Class) |
| GameplayTagContainer für Emotionen | ❌ (Key-Value-Metadata) | ✅ |
| Parallele Conversations | ✅ | ❌ (ein aktiver Dialog) |
| Replication-Pattern | ✅ | ✅ (Architektur-Ziel) |
| Multiplayer-UX | ✅ | ❌ (Non-Goal) |

## Komplementäre Projekte

* **CC-Stärken**: Multiplayer-Orchestrierung, mehrere parallele Conversations, Integration mit Epic-intern Lyra-Code-Stack.
* **MayDialogue-Stärken**: Designer-First-Editor, Built-in UI/Audio/Kamera, GAS-Integration, In-Editor-Preview.

## Wann solltest du CC statt MayDialogue nutzen?

* Dein Projekt ist **Multiplayer-First** mit komplexer Choice-Voting-Mechanik.
* Du bist tief im Lyra-Code-Stack.
* Du willst minimalistisch und hast eigene UI/Audio-Pipeline.

## Wann solltest du MayDialogue nutzen?

* Du willst ein **fertiges Dialogsystem in Stunden**, nicht Wochen.
* Du baust **Horror-, VN-, RPG-Singleplayer**.
* Du nutzt **GAS** für Character-Stats und willst Dialoge daran knüpfen.
* **Designer-Erlebnis** ist dir wichtig (Inline-Edit, Sprecher-Farben, In-Editor-Preview).

## Kann ich beides kombinieren?

Technisch ja — beide sind Plugins, die unterschiedliche Module mitbringen. Aber:

* **Namenskonflikte** möglich (ähnliche Klassen wie `ConversationInstance` vs. `MayDialogueInstance` unterscheiden sich, haben aber ähnliche Zwecke).
* **Kein gemeinsames Datenmodell**. Ein CC-Asset kann nicht in MayDialogue geöffnet werden und umgekehrt.

Empfehlung: **Ein System pro Projekt** wählen. Wenn du bestehende CC-Assets migrieren willst: manuell nach MayDialogue portieren (der Struktur-Unterschied ist überschaubar).

## Code-Parallelen

Wenn du CC-Code kennst, findest du dich in MayDialogue schnell zurecht:

| CC | MayDialogue |
| --- | --- |
| `UConversationInstance` | `UMayDialogueInstance` |
| `UConversationParticipantComponent` | `UMayDialogueParticipant` |
| `FConversationTaskResult` | `FMayDialogueTaskResult` |
| `EConversationTaskResultType` | `EMayDialogueTaskResultType` |
| `FClientConversationMessage` | `FMayDialogueMessage` |
| `FClientConversationOptionEntry` | `FMayDialogueChoiceEntry` |
| `UConversationRegistry` | (nicht benötigt; Asset direkt referenziert) |
| `FConversationBranchPoint` | `FMayDialogueBranchPoint` |

Die **Konzepte** sind 1:1. Die **APIs** leicht unterschiedlich (bessere Lesbarkeit, tighter Naming).
