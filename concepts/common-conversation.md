# Verhältnis zu CommonConversation

Epic liefert in UE 5.7 ein internes Plugin namens **CommonConversation** (CC), das in Lyra und anderen Epic-Projekten eingesetzt wird. MayDialogue basiert gedanklich und teilweise code-technisch darauf. Dieses Kapitel macht transparent, **was übernommen wurde, was verändert wurde, und warum keine externe CC-Dependency existiert**.

## Kernidee: Inspiration, keine Dependency

**MayDialogue zieht das CommonConversation-Plugin nicht als Abhängigkeit ein.** Der relevante Code lebt in MayDialogues eigenen Modulen – teilweise als angepasster Fork von CC-Code, teilweise konzeptionell übernommen. Wer MayDialogue installiert, braucht CC **nicht** zu aktivieren.

Gründe dafür:

1. **CC ist als Epic-interne Werkstoff-Schicht markiert** und kann ohne Vorwarnung geändert werden.
2. **CC bringt minimale UI-/Audio-Integration** – MayDialogue liefert deutlich mehr und braucht saubere Kontrolle über seine eigene Pipeline.
3. **Einfachere Installation** – eine einzige Plugin-Kopie, ein einziges Modul-Set.

## Was übernommen wurde

### Instance-Lifecycle-Modell

CC's `UConversationInstance` ist ein reines `UObject`, das das laufende Gespräch kapselt (kein Actor, keine Komponente). Wir übernehmen das 1:1 als [`UMayDialogueInstance`](instance-lifecycle.md).

### Participant-Komponenten-Modell

CC's `UConversationParticipantComponent` wird zu `UMayDialogueParticipant`. Wir ergänzen Portraits, DefaultDialogue, PersistentMemory, Kamera-Offsets, Audio-Overrides.

### Sub-Node-Komposition

CC's TaskNode trägt Requirements, Choices und SideEffects als Sub-Nodes. Das übernehmen wir vollständig (siehe [Graph & visuelle Sprache](graph-visual-language.md)).

### Task-Result-Enum

CC's `FConversationTaskResult` mit Enum (Advance/Abort/Pause/Return-to-Start) ist ein sauberes deklaratives Pattern. MayDialogue spiegelt das als [`FMayDialogueTaskResult`](instance-lifecycle.md#node-ausfuhrung) mit `EMayDialogueTaskResultType`.

### Scope-Stack für Sub-Conversations

Link-Nodes pushen auf den Stack, Exits poppen. Robustes Return-Modell – bleibt unverändert.

### Replikations-Pattern als Architektur-Ziel

Server-/Client-RPCs, NetSerialize-Handles, ReplicatedUsing-Properties. Die Infrastruktur ist angelegt, Phase 2 füllt sie.

### Scan-and-Recompile-Mechanik

Bei Asset-Änderungen läuft der Compiler automatisch. Bleibt als internes Editor-Feature erhalten.

## Was verändert wurde

### Ein Entry-Point pro Asset statt viele

CC erlaubt mehrere Entry-Tags (Greeting, Farewell, Angry). Wir entschieden uns für **genau einen Entry pro Asset**. Wenn du verschiedene Start-Varianten brauchst: separate Assets oder am Anfang einen Branch.

**Begründung**: Übersichtlichkeit schlägt Flexibilität. Mehrere Entry-Points verkomplizieren das Asset ohne praktischen Vorteil.

### Eigener visueller Graph

CC nutzt UE's `AIGraph`-Basis (dieselbe wie BehaviorTree-Editor). Wir bauen einen **Custom-EdGraph** (`UMayDialogueGraph`), damit wir die visuelle Sprache (Sprecher-Farben, Inline-Texte, Sub-Node-Rendering) voll kontrollieren.

### Mitgelieferte UI

CC liefert nur ein Daten-Modell (`FClientConversationMessage`). MayDialogue liefert [Slate-Debug-Widget + komponenten-basiertes UMG](../ui/README.md).

### Mitgelieferte Audio-Pipeline

CC hat keine Audio-Integration. MayDialogue bringt [3D-First-Audio, 3-Level-Fallback, Babel-System](../audio/README.md).

### Reiche vordefinierte Nodes

CC liefert nur abstrakte Basis-Klassen (TaskNode, ChoiceNode, RequirementNode, SideEffectNode). MayDialogue liefert **19 konkrete Nodes** (siehe [Node-Referenz](../nodes/README.md)).

### Aktionen als Action-Nodes UND SideEffect-Sub-Nodes

CC bietet Aktionen nur als SideEffect-Sub-Nodes im Body eines TaskNodes. MayDialogue bietet denselben Aktionstyp **in beiden Formen**:

* Als eigenständige **Action-Node** im Graph, wenn die Aktion der Hauptschritt ist.
* Als **SideEffect-Sub-Node**, wenn sie nebenbei beim Betreten eines Nodes mitläuft.

Designer wählt pro Situation – Sichtbarkeit und Kompaktheit sind keine Gegensätze mehr.

### GameplayTagContainer für Emotionen

CC hat `FClientConversationMessage.MetadataParameters` als generisches Key-Value-String-Array. Wir nutzen [GameplayTagContainer](emotions-tags.md) – typisiert, hierarchisch, mehrfach.

### Typewriter-Engine mit Rich-Text-Tags

In CC nicht vorhanden. MayDialogue liefert Pause/Speed-Tags im Parser plus Shake/Wave/Color/Bold als Rich-Text-Decorators.

### Kamera-Nodes

In CC nicht vorhanden. MayDialogue liefert CameraFocus, CameraShake.

### In-Editor-Preview ohne PIE

In CC nicht vorhanden (nur PIE-Debugger). MayDialogues [Preview-Runner](../editor/preview-runner.md) spielt Dialoge direkt im Asset-Editor.

### Ein einziger aktiver Dialog erzwungen

CC erlaubt mehrere parallele Conversations. MayDialogue erzwingt **genau einen aktiven Dialog pro Welt** – macht UI, Audio-Handling und Input-Management deutlich einfacher.

### GAS-Integration als erste-Klasse-Bürger

CC hat keine GAS-Bindings. MayDialogue liefert [voll integrierte GAS-Requirements und -Aktions-Nodes](../gas/README.md).

## Gemeinsame Begriffe

Zur Orientierung, wenn du CC-Code oder -Dokumentation querliest:

| CC-Begriff | MayDialogue-Entsprechung |
| --- | --- |
| `UConversationInstance` | `UMayDialogueInstance` |
| `UConversationParticipantComponent` | `UMayDialogueParticipant` |
| `UConversationRegistry` | *(nicht benötigt, wir arbeiten mit `UMayDialogueAsset` direkt)* |
| `FConversationTaskResult` | `FMayDialogueTaskResult` |
| `FClientConversationMessage` | `FMayDialogueMessage` |
| `FClientConversationOptionEntry` | `FMayDialogueChoiceEntry` |

## Wann du CC statt MayDialogue nutzen solltest

| Szenario | Empfehlung |
| --- | --- |
| Multiplayer-First-Game mit komplexer Choice-Voting-Mechanik | CC (mehrere Entry-Tags, parallele Conversations). |
| Sehr großes Projekt mit etabliertem Lyra-Code-Stack | CC (Integration bereits da). |
| Indie-/Horror-/Singleplayer-Projekt, das **fertige UI, Audio, Typewriter, Kamera** will | MayDialogue. |
| Projekt, das GAS nativ in Dialogen nutzen will | MayDialogue. |
| Projekt, das Designer-Friendly-Graph in Vordergrund stellt | MayDialogue. |

## Zusammengefasst

MayDialogue ist **nicht** CC – aber es verdankt CC viele Ideen. Die Laufzeit-Architektur ist robust, replikationsfähig und aus bewährten Mustern destilliert. Die Bedien-Oberfläche, die Audio-Pipeline, die UI-Schicht und die GAS-Integration sind Neubauten, die CC nicht mitbringt.

Ende der Kern-Konzepte. Weiter mit dem **Editor**: [Asset-Editor](../editor/README.md).
