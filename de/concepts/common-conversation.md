---
description: MayDialogue und CommonConversation — Inspiration, Unterschiede, und wann du was wählen solltest.
---

# MayDialogue & CommonConversation

Epic Games liefert in Unreal Engine 5 ein internes Plugin namens **CommonConversation** (CC), das im Lyra Starter Game und anderen Epic-Projekten eingesetzt wird. MayDialogue hat seine Runtime-Architektur bewusst an CC orientiert — übernimmt die richtigen Ideen, ersetzt die Lücken durch eine vollständige Schicht darüber, und verzichtet auf jede Abhängigkeit zum CC-Plugin.

Dieses Kapitel macht transparent: Was ist CommonConversation? Was hat MayDialogue davon übernommen und warum? Was fügt MayDialogue hinzu? Und wann solltest du CC statt MayDialogue erwägen?

> 📸 **Bild-Platzhalter:** `cc-comparison-overview.png` — Nebeneinander-Diagramm CC vs. MayDialogue.
> *Setup:* Zwei Spalten. Links „CommonConversation": Boxen für Instance, Participant, Task-Result-Loop, Sub-Node-Komposition — aber keine Verbindungslinien zu UI, Audio, Editor. Rechts „MayDialogue": Dieselben vier Kern-Boxen (hervorgehoben), plus Verbindungslinien zu Graph-Editor, UI-Schicht (Slate + UMG), Audio-Pipeline (3D + Babel), Kamera-Nodes, GAS-Integration, Persistence, Debugger/Preview. Titel: „Gleiches Fundament — vollständiger Bau."

## Was CommonConversation ist

CommonConversation ist Epics interne Dialogue-Runtime — ein Datenmodell und Laufzeit-Kern für server-autoritative, replizierbare Gespräche. Es ist in Lyra und Epic-internen Projekten als **Werkstoff-Schicht** konzipiert: robuste Architektur-Fundamente, absichtlich ohne fertige UI, Audio oder Editor-UX.

CC beantwortet die Frage: „Wie strukturiert man ein Gespräch als Server-Object mit sauberem Replication-Pattern?" Es beantwortet nicht: „Wie sieht der Dialog für den Spieler aus, wie klingt er, und wie baue ich ihn im Editor?"

**CC ist kein Endprodukt für Käufer** — es ist ein Architektur-Muster, das Epic für eigene Projekte nutzt. Es ist nicht auf dem Marketplace verfügbar und nicht für den direkten Einsatz ohne eigenen Wrapper gedacht.

## Warum keine externe CC-Abhängigkeit

**MayDialogue zieht CommonConversation nicht als Abhängigkeits-Plugin ein.** Die übernommenen Muster leben in MayDialogues eigenen Modulen — als konzeptionelle Übernahmen und angepasste Implementierungen. Wer MayDialogue installiert, muss CC nicht aktivieren, kennen oder konfigurieren.

Drei Gründe:

1. **CC ist als Epic-interne Werkstoff-Schicht markiert** — ohne Stabilitätsversprechen und ohne Vorwarnung bei Änderungen.
2. **CC bietet keine UI/Audio-Integration** — MayDialogue braucht vollständige Kontrolle über seine eigene Pipeline.
3. **Einfachere Installation** — ein Plugin, ein Modul-Set, keine versteckten Abhängigkeiten.

Die Attribution ist klar geregelt: MayDialogue inspiriert sich architektonisch an CC, kopiert keinen Epic-Quellcode, und hält alle Rechte im eigenen Plugin (→ `CREDITS.md`).

## Was MayDialogue von CC übernommen hat

### Server-autoritative Instanz

CCs `UConversationInstance` ist ein reines `UObject` — kein Actor, keine Komponente — das das laufende Gespräch kapselt und nur auf dem Server existiert. MayDialogue spiegelt dieses Muster 1:1 als `UMayDialogueInstance`.

Das Muster ist bewusst gewählt: Gespräche enthalten spielrelevante State-Übergänge (Variablen, Quest-Fortschritt, Choice-Auswahl). Diese müssen server-autoritativ sein, damit Clients sie nicht fälschen können. Clients sehen den Gesprächsstand über einen Client-RPC-Mirror; Eingaben gehen über Server-RPCs.

### Participant-Komponentenmodell

CCs `UConversationParticipantComponent` verbindet Actors im Level mit Einträgen im Gesprächs-Asset über einen Tag. MayDialogue übernimmt dieses Modell als `UMayDialogueParticipant` und erweitert es um Portraits, persistentes Gedächtnis (`PersistentMemory`), Kamera-Offset-Ankerpunkte, Audio-Overrides und eine `DefaultDialogue`-Referenz.

### Deklarative Task-Result-Nodes

CCs Task-Result-Pattern ist elegant: Nodes **beschreiben**, was sie tun, und **geben zurück**, wie der Dialog weiterlaufen soll — als Enum-Wert (`FConversationTaskResult`). Der Traversal-Loop in der Instance **interpretiert** das Ergebnis. Kein Node steuert die State-Machine selbst.

MayDialogue spiegelt dieses Muster als `FMayDialogueTaskResult` mit `EMayDialogueTaskResultType` (Advance / Pause+Choices / Abort, plus experimentelle Return-Werte). Die zentrale Invariante bleibt dieselbe: **Nodes deklarieren, die Instance entscheidet.**

### Sub-Node-Komposition

CC's TaskNode-Konzept: Ein Node trägt Requirements (darf ich ausgeführt werden?), Choices (was kann der Spieler wählen?) und SideEffects (was passiert nebenbei?) als eingebettete Sub-Objekte. MayDialogue übernimmt diese Komposition vollständig — Requirements, Choices und SideEffects als kompakte Pills im Node-Body, nicht als separate Graph-Boxen.

### Scope-Stack für Sub-Gespräche

CCs Link-/Sub-Conversation-Mechanik: Ein Link-Node wechselt in einen anderen Gesprächs-Kontext, merkt sich den Rücksprung-Punkt auf einem Stack, und poppt ihn wenn der verlinkte Kontext endet. MayDialogue übernimmt dieses Modell für `Link`- und `SubGraph`-Nodes — tiefe Verschachtelung ohne manuellen Rücksprung-Code.

## Was MayDialogue hinzufügt

### Visueller Graph-Editor

CC hat keinen eigenen Asset-Editor. MayDialogue liefert einen vollständigen Custom-EdGraph (`UMayDialogueGraph`): Sprecher-Farben in Title-Bars, Inline-Texte, Doppelklick-Inline-Editing, Requirement/SideEffect-Pills, Palette mit Drag-and-Drop, Outline-Panel, Find-in-Dialogue, Auto-Layout, SubGraph-Breadcrumbs, Kommentar-Boxen, Reroute-Knots. **Der Graph liest sich wie ein Drehbuch** — das ist das primäre Design-Ziel des Editors.

### Fertige UI in zwei Schichten

CC liefert nur `FClientConversationMessage` — ein Daten-Payload. Was damit passiert, ist Projektsache.

MayDialogue liefert zwei produktionsreife UI-Schichten:

- **Slate-Debug-Widget** (`SMayDialogueWidget`): Auto-spawn, kein Setup, funktioniert sofort — für Prototypen und Tests.
- **UMG-Komponentensystem** (`UMayDialogueWidget`): Sechs einzeln austauschbare Komponenten-Widgets (DialogFrame, Speaker, Text, ChoiceList, ChoiceButton, SkipButton), Typewriter-Engine, Rich-Text-Decorators (`<shake>`, `<wave>`, `<color>`, `<b>`), drei mitgelieferte Themes (Horror / Visual Novel / RPG).

### 3D-Audio-Pipeline und Babel

CC hat keine Audio-Integration. MayDialogue liefert:

- **Vier-stufige Fallback-Kette** (Plugin-Settings → Speaker → Participant → Node) für Spatial-3D- und Force-2D-Wiedergabe.
- **Babel-Synthstimmen** (`UMayDialogueBabelSynth`): Prozedurale Platzhalter-Stimmen für jeden NPC — in der Entwicklung hörst du das Timing sofort, ohne echtes VO. Shippable als Stil-Option (Fears-to-Fathom-artige Blip-Stimmen). Profile als Data Assets pro Sprecher konfigurierbar.
- **VoicePerCulture-Map** pro SayLine: Mehrsprachige VO mit BCP-47-Fallback-Kette.

### Kamera-Nodes

CC hat keine Kamera-Integration. MayDialogue liefert:

- **CameraFocus** (drei Modi: SpeakerLook, CineCamera-Anchor per Tag, LevelSequence mit optionalem Wait) — schnelle Dialog-Regie direkt im Graph.
- **CameraShake** (`UCameraShakeBase`, Skalierung, räumlicher Falloff).
- Auto-Focus-Speaker und Auto-Face-Partner als Opt-ins auf der Participant-Komponente.

### In-Editor-Preview und PIE-Debugger

CC hat keinen In-Editor-Preview (nur PIE-Debugging). MayDialogue liefert:

- **Preview Runner**: Kompletten Dialog-Durchlauf direkt im Asset-Editor — ohne PIE zu starten. Typewriter, Voice/Babel, Choices, Variablen live änderbar.
- **PIE-Debugger**: Breakpoints pro Node (persistent), Step-Controls (Over/Into/Out auch in Sub-Graphs), Active-Node-Highlight, Variable-Watch mit Schreibzugriff.

### GAS-Integration als erste Klasse

CC hat keine GAS-Bindings. MayDialogue liefert voll integrierte GAS-Requirements (HasTag, HasAbility, CheckAttribute), GAS-Actions/SideEffects (AddTag, RemoveTag, ApplyEffect, TriggerCue) und Lifecycle-Cue-Bindings als No-Code-GAS-Brücke — isoliert im `MayDialogueGAS`-Modul, aber automatisch mitgeladen.

### Persistenz und Variablen

CC hat keine eingebaute Persistenz. MayDialogue liefert:

- **Dialog-Scope-Variablen** (`FInstancedPropertyBag`): Temporäre Variablen, die im laufenden Gespräch existieren.
- **Participant-Persistent-Memory** (`PersistentMemory`, SaveGame-geflaggt): Variablen, die Gespräche überdauern und automatisch im SaveGame landen.
- **QuickSave-Helper** (`UMayDialogueSaveHelper`) für Projekte ohne eigenes Save-System.

### 19 konkrete Nodes statt abstrakter Basisklassen

CC liefert nur abstrakte Basisklassen (TaskNode, ChoiceNode, RequirementNode, SideEffectNode). MayDialogue liefert 19 fertige, sofort nutzbare Nodes (9 Core-Nodes + 10 Action-Nodes mit vollständiger SideEffect-Parität) — und jede Basisklasse ist Blueprint- und C++-subclassbar für eigene Node-Typen.

## Ein bewusster Unterschied: ein Entry-Point pro Asset

CC erlaubt mehrere Entry-Tags pro Conversation (Greeting, Farewell, Angry — je nach Kontext wird ein anderer Tag angesteuert). MayDialogue hat **genau einen Entry-Node pro Asset**.

**Begründung**: Übersichtlichkeit schlägt Flexibilität. Ein Asset, ein klarer Einstiegspunkt — der Graph liest sich linear. Wenn du verschiedene Start-Kontexte brauchst, verwendest du separate Assets oder einen Branch direkt nach dem Entry.

## Begriff-Übersetzungstabelle

Wenn du CC-Quellcode oder Lyra-Dokumentation querliest und MayDialogue-Entsprechungen suchst:

| CommonConversation | MayDialogue-Entsprechung |
| --- | --- |
| `UConversationInstance` | `UMayDialogueInstance` |
| `UConversationParticipantComponent` | `UMayDialogueParticipant` |
| `UConversationRegistry` | *(nicht benötigt — Assets werden direkt referenziert)* |
| `FConversationTaskResult` | `FMayDialogueTaskResult` |
| `FClientConversationMessage` | `FMayDialogueMessage` |
| `FClientConversationOptionEntry` | `FMayDialogueChoiceEntry` |
| `UConversationNode` | `UMayDialogueNode_Base` |
| `UConversationRequirementNode` | `UMayDialogueRequirement` |
| `UConversationSideEffectNode` | `UMayDialogueSideEffect` |

## Wann du CommonConversation statt MayDialogue erwägen solltest

MayDialogue ist die richtige Wahl für die überwiegende Mehrheit der Projekte. Es gibt jedoch Konstellationen, in denen CC-nativ sinnvoller ist:

| Szenario | Empfehlung |
| --- | --- |
| Du arbeitest in einem Lyra-basierten Projekt mit bestehendem CC-Stack und einem dedizierten Programmer-Team | CC (kein Aufwand, Stack bereits integriert) |
| Du brauchst mehrere Entry-Tags pro Conversation ohne separate Assets | CC (Multi-Entry ist CC-Feature) |
| Du baust ein Multiplayer-Spiel mit Choice-Voting-Mechanik für mehrere Spieler gleichzeitig | CC (Choice-Voting und parallele Conversations sind CC-Kernfeatures) |
| Du willst einen Dialog in 5 Minuten spielbar — ohne Boilerplate, ohne Widget-Bau, ohne Audio-Setup | MayDialogue |
| Dein Projekt braucht GAS-Requirements und -Aktionen in Dialogen | MayDialogue |
| Du willst fertige UI-Themes, Typewriter, Babel-Stimmen und Kamera-Regie | MayDialogue |
| Du bist Allein-Entwickler oder kleines Team ohne dedizierten Engine-Programmer | MayDialogue |
| Dein Projekt ist kein Lyra-Fork | MayDialogue (keine gemeinsame Basis vorhanden) |

{% hint style="info" %}
MayDialogue ist kein CC-Wrapper und keine CC-Alternative — es ist ein eigenständiges, vollständiges Dialogsystem, das **dieselben bewährten Architektur-Muster** verwendet und darüber einen vollständigen Bau setzt, den CC bewusst offen lässt.
{% endhint %}

## Zusammenfassung

MayDialogue verdankt CommonConversation einige seiner wichtigsten Architektur-Ideen: die server-autoritative Instance, das Participant-Komponenten-Modell, das deklarative Task-Result-Pattern und die Sub-Node-Komposition. Diese Fundamente sind solide, in Lyra-Projekten erprobt und nicht neu erfunden.

Was MayDialogue darüber baut — der visuelle Editor, die UI-Schichten, die Audio-Pipeline, Babel-Stimmen, Kamera-Nodes, GAS-Integration, Persistence und das Debugging-Tooling — ist eigenständige Arbeit, die CC bewusst offen lässt.

Weiter: [Graph & visuelle Sprache](graph-visual-language.md).
