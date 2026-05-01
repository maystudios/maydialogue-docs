# Table of Contents

* [Willkommen bei MayDialogue](README.md)

## Erste Schritte

* [Einführung](getting-started/README.md)
* [Installation](getting-started/installation.md)
* [Quick Start – Dein erster Dialog in 5 Minuten](getting-started/quick-start.md)
* [Ein vollständiger Walkthrough](getting-started/first-dialogue.md)
* [Projekt-Einstellungen](getting-started/project-settings.md)

## Kern-Konzepte

* [Überblick](concepts/README.md)
* [Architektur im Großen](concepts/architecture.md)
* [Graph & visuelle Sprache](concepts/graph-visual-language.md)
* [Instance & Lifecycle](concepts/instance-lifecycle.md)
* [Participants & Sprecher](concepts/participants-speakers.md)
* [Variablen & Scopes](concepts/variables-scopes.md)
* [Emotionen & Tag-Container](concepts/emotions-tags.md)

## Der Editor

* [Überblick](editor/README.md)
* [Der Asset-Editor](editor/asset-editor.md)
* [Das Graph-Panel](editor/graph-panel.md)
* [Speakers-Panel](editor/speakers-panel.md)
* [Variables-Panel](editor/variables-panel.md)
* [Outline](editor/outline.md)
* [Find-in-Dialogue](editor/find.md)
* [Preview Runner – ohne PIE testen](editor/preview-runner.md)
* [Debugger & Breakpoints](editor/debugger.md)
* [Komfort-Features](editor/comfort-features.md)

## Node-Referenz

* [Überblick](nodes/README.md)
* [Core-Nodes](nodes/core/README.md)
  * [Entry](nodes/core/entry.md)
  * [Exit](nodes/core/exit.md)
  * [Say Line](nodes/core/say-line.md)
  * [Player Choice](nodes/core/player-choice.md)
  * [Branch](nodes/core/branch.md)
  * [Random Line](nodes/core/random-line.md)
  * [Wait](nodes/core/wait.md)
  * [Link](nodes/core/link.md)
  * [SubGraph](nodes/core/sub-graph.md)
* [Action-Nodes](nodes/actions/README.md)
  * [Camera Focus](nodes/actions/camera-focus.md)
  * [Camera Shake](nodes/actions/camera-shake.md)
  * [Play Animation](nodes/actions/play-animation.md)
  * [Apply Effect](nodes/actions/apply-effect.md)
  * [Set Variable](nodes/actions/set-variable.md)
  * [Fire Event](nodes/actions/fire-event.md)
  * [Play Sound](nodes/actions/play-sound.md)
  * [Add Tag](nodes/actions/add-tag.md)
  * [Remove Tag](nodes/actions/remove-tag.md)
  * [Trigger Cue](nodes/actions/trigger-cue.md)
* [Sub-Nodes](nodes/sub-nodes/README.md)
  * [Requirement](nodes/sub-nodes/requirement.md)
  * [Choice](nodes/sub-nodes/choice.md)
  * [SideEffect](nodes/sub-nodes/side-effect.md)

## Runtime-Integration

* [Überblick](runtime/README.md)
* [Einen Dialog starten](runtime/starting-dialogues.md)
* [Subsystem-API](runtime/subsystem-api.md)
* [Blueprint-Library](runtime/library-api.md)
* [Bridge & Lifecycle-Events](runtime/bridge-events.md)
* [Read/Write-API](runtime/read-write-api.md)

## UI-System

* [Überblick](ui/README.md)
* [Slate-Debug-Widget](ui/slate-debug-widget.md)
* [UMG-Architektur](ui/umg-architecture.md)
* [Dialog Frame](ui/dialog-frame.md)
* [Speaker Widget](ui/speaker-widget.md)
* [Text Widget](ui/text-widget.md)
* [Choice List & Choice Button](ui/choice-list.md)
* [Skip Button](ui/skip-button.md)
* [Typewriter-Engine](ui/typewriter.md)
* [Rich-Text-Tags](ui/rich-text-tags.md)
* [Themes & Starterkits](ui/themes.md)

## Audio-System

* [Überblick](audio/README.md)
* [Drei-Ebenen-Fallback](audio/three-level-fallback.md)
* [Speaker-Overrides](audio/speaker-overrides.md)
* [Node-Overrides](audio/node-overrides.md)
* [Lokalisierung (VoicePerCulture)](audio/localization.md)
* [Babel-System](audio/babel-system.md)
* [Babel-Profile](audio/babel-profiles.md)

## GAS-Integration

* [Überblick](gas/README.md)
* [Requirements](gas/requirements.md)
* [Aktionen](gas/actions.md)
* [Helpers](gas/helpers.md)
* [Eigene GAS-Nodes erstellen](gas/extending.md)

## Persistence

* [Überblick](persistence/README.md)
* [SaveGame-Integration](persistence/save-integration.md)
* [Participant-Memory](persistence/participant-memory.md)
* [QuickSave-Helper](persistence/quicksave-helper.md)

## Erweiterung

* [Überblick](extension/README.md)
* [Eigene Nodes per Blueprint](extension/custom-nodes.md)
* [Eigene Requirements](extension/custom-requirements.md)
* [Eigene SideEffects](extension/custom-side-effects.md)
* [Bridge implementieren](extension/bridge-implementation.md)
* [C++-Erweiterung — Grundlagen](extension/cpp-fundamentals.md)

## Rezepte

* [Überblick](recipes/README.md)
* [Einfaches NPC-Gespräch](recipes/simple-npc-talk.md)
* [Zufällige Begrüßungen](recipes/random-greetings.md)
* [Verzweigungen mit Bedingungen](recipes/branching-conditions.md)
* [Wiederverwendbare Dialog-Fragmente](recipes/linking-dialogues.md)
* [SubGraph-Organisation](recipes/subgraph-organization.md)
* [Choice nur sichtbar mit Tag](recipes/choice-with-tag-requirement.md)
* [Choice mit Attribut-Bedingung](recipes/choice-with-attribute-requirement.md)
* [GAS-getriebener Dialog](recipes/gas-driven-dialogue.md)
* [GameplayEffect aus Dialog anwenden](recipes/apply-gameplay-effect.md)
* [Quest-Status im Dialog lesen](recipes/quest-status-in-dialogue.md)
* [Dialog setzt Quest-Progress](recipes/dialogue-sets-quest-progress.md)
* [Beziehungs-Score tracken](recipes/relationship-counter.md)
* [NPC erinnert sich beim 2. Treffen](recipes/npc-remembers-meeting.md)
* [Mehrsprachigkeit](recipes/multilingual-dialogue.md)
* [Innerer Monolog mit 2D-Audio](recipes/inner-monologue-2d.md)
* [Kamera-Schwenk auf Sprecher](recipes/camera-pan-on-speaker.md)
* [Jump-Scare mit Camera-Shake](recipes/jump-scare-shake.md)
* [NPC-Animation während Zeile](recipes/npc-animation-during-line.md)
* [Timed Choice (Auto-Auswahl)](recipes/timed-choice.md)
* [Wait auf externes GameplayEvent](recipes/wait-for-event.md)
* [Eigenes UMG-Widget anbinden](recipes/custom-umg-widget.md)
* [Eigenen Requirement in Blueprint bauen](recipes/custom-blueprint-requirement.md)

## Referenz

* [Überblick](reference/README.md)
* [Projekt-Einstellungen](reference/project-settings.md)
* [Editor-Einstellungen](reference/editor-settings.md)
* [MayDialogueLibrary](reference/api-library.md)
* [MayDialogueSubsystem](reference/api-subsystem.md)
* [Delegates & Events](reference/api-delegates.md)
* [Typen & Enums](reference/types.md)

## Troubleshooting

* [Überblick](troubleshooting/README.md)
* [Häufige Probleme](troubleshooting/common-issues.md)
* [Debug-Tipps](troubleshooting/debugging-tips.md)
* [Bekannte Issues](troubleshooting/known-issues.md)

## Anhang

* [Roadmap](appendix/roadmap.md)
* [Glossar](appendix/glossary.md)
