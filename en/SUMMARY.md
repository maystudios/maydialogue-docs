# Table of Contents

* [Welcome to MayDialogue](README.md)

## Getting Started

* [Introduction](getting-started/README.md)
* [Installation](getting-started/installation.md)
* [Quick Start – Your First Dialogue in 5 Minutes](getting-started/quick-start.md)
* [A Complete Walkthrough](getting-started/first-dialogue.md)
* [Project Settings](getting-started/project-settings.md)

## Core Concepts

* [Overview](concepts/README.md)
* [Architecture Overview](concepts/architecture.md)
* [Graph & Visual Language](concepts/graph-visual-language.md)
* [Instance & Lifecycle](concepts/instance-lifecycle.md)
* [Participants & Speakers](concepts/participants-speakers.md)
* [Variables & Scopes](concepts/variables-scopes.md)
* [Emotions & Tag Containers](concepts/emotions-tags.md)

## The Editor

* [Overview](editor/README.md)
* [The Asset Editor](editor/asset-editor.md)
* [The Graph Panel](editor/graph-panel.md)
* [Speakers Panel](editor/speakers-panel.md)
* [Variables Panel](editor/variables-panel.md)
* [Outline](editor/outline.md)
* [Find-in-Dialogue](editor/find.md)
* [Preview Runner – Test Without PIE](editor/preview-runner.md)
* [Debugger & Breakpoints](editor/debugger.md)
* [Comfort Features](editor/comfort-features.md)

## Node Reference

* [Overview](nodes/README.md)
* [Core Nodes](nodes/core/README.md)
  * [Entry](nodes/core/entry.md)
  * [Exit](nodes/core/exit.md)
  * [Say Line](nodes/core/say-line.md)
  * [Player Choice](nodes/core/player-choice.md)
  * [Branch](nodes/core/branch.md)
  * [Random Line](nodes/core/random-line.md)
  * [Wait](nodes/core/wait.md)
  * [Link](nodes/core/link.md)
  * [SubGraph](nodes/core/sub-graph.md)
* [Action Nodes](nodes/actions/README.md)
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

## Runtime Integration

* [Overview](runtime/README.md)
* [Starting a Dialogue](runtime/starting-dialogues.md)
* [Subsystem API](runtime/subsystem-api.md)
* [Blueprint Library](runtime/library-api.md)
* [Bridge & Lifecycle Events](runtime/bridge-events.md)
* [Read/Write API](runtime/read-write-api.md)

## UI System

* [Overview](ui/README.md)
* [Slate Debug Widget](ui/slate-debug-widget.md)
* [UMG Architecture](ui/umg-architecture.md)
* [Dialog Frame](ui/dialog-frame.md)
* [Speaker Widget](ui/speaker-widget.md)
* [Text Widget](ui/text-widget.md)
* [Choice List & Choice Button](ui/choice-list.md)
* [Skip Button](ui/skip-button.md)
* [Typewriter Engine](ui/typewriter.md)
* [Rich Text Tags](ui/rich-text-tags.md)
* [Themes & Starter Kits](ui/themes.md)

## Audio System

* [Overview](audio/README.md)
* [Three-Level Fallback](audio/three-level-fallback.md)
* [Speaker Overrides](audio/speaker-overrides.md)
* [Node Overrides](audio/node-overrides.md)
* [Localization (VoicePerCulture)](audio/localization.md)
* [Babel System](audio/babel-system.md)
* [Babel Profiles](audio/babel-profiles.md)

## GAS Integration

* [Overview](gas/README.md)
* [Requirements](gas/requirements.md)
* [Actions](gas/actions.md)
* [Helpers](gas/helpers.md)
* [Creating Custom GAS Nodes](gas/extending.md)

## Persistence

* [Overview](persistence/README.md)
* [SaveGame Integration](persistence/save-integration.md)
* [Participant Memory](persistence/participant-memory.md)
* [QuickSave Helper](persistence/quicksave-helper.md)

## Extension

* [Overview](extension/README.md)
* [Custom Nodes via Blueprint](extension/custom-nodes.md)
* [Custom Requirements](extension/custom-requirements.md)
* [Custom SideEffects](extension/custom-side-effects.md)
* [Implementing the Bridge](extension/bridge-implementation.md)
* [C++ Extension — Fundamentals](extension/cpp-fundamentals.md)

## Recipes

* [Overview](recipes/README.md)
* [Simple NPC Conversation](recipes/simple-npc-talk.md)
* [Random Greetings](recipes/random-greetings.md)
* [Branching with Conditions](recipes/branching-conditions.md)
* [Reusable Dialogue Fragments](recipes/linking-dialogues.md)
* [SubGraph Organization](recipes/subgraph-organization.md)
* [Choice Visible Only With Tag](recipes/choice-with-tag-requirement.md)
* [Choice with Attribute Condition](recipes/choice-with-attribute-requirement.md)
* [GAS-Driven Dialogue](recipes/gas-driven-dialogue.md)
* [Apply a GameplayEffect from Dialogue](recipes/apply-gameplay-effect.md)
* [Reading Quest Status in Dialogue](recipes/quest-status-in-dialogue.md)
* [Dialogue Sets Quest Progress](recipes/dialogue-sets-quest-progress.md)
* [Tracking a Relationship Score](recipes/relationship-counter.md)
* [NPC Remembers the Second Meeting](recipes/npc-remembers-meeting.md)
* [Multilingual Dialogue](recipes/multilingual-dialogue.md)
* [Inner Monologue with 2D Audio](recipes/inner-monologue-2d.md)
* [Camera Pan to Speaker](recipes/camera-pan-on-speaker.md)
* [Jump Scare with Camera Shake](recipes/jump-scare-shake.md)
* [NPC Animation During a Line](recipes/npc-animation-during-line.md)
* [Timed Choice (Auto-Select)](recipes/timed-choice.md)
* [Wait for an External GameplayEvent](recipes/wait-for-event.md)
* [Connecting a Custom UMG Widget](recipes/custom-umg-widget.md)
* [Building a Custom Requirement in Blueprint](recipes/custom-blueprint-requirement.md)

## Reference

* [Overview](reference/README.md)
* [Project Settings](reference/project-settings.md)
* [Editor Settings](reference/editor-settings.md)
* [MayDialogueLibrary](reference/api-library.md)
* [MayDialogueSubsystem](reference/api-subsystem.md)
* [Delegates & Events](reference/api-delegates.md)
* [Types & Enums](reference/types.md)

## Troubleshooting

* [Overview](troubleshooting/README.md)
* [Common Issues](troubleshooting/common-issues.md)
* [Debugging Tips](troubleshooting/debugging-tips.md)
* [Known Issues](troubleshooting/known-issues.md)

## Appendix

* [Roadmap](appendix/roadmap.md)
* [Glossary](appendix/glossary.md)
