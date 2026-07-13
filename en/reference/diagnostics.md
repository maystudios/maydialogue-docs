---
description: "Stable MayDialogue validator rule IDs, what each finding means, and how to fix it."
---

# Validator Diagnostics

The compiler and MCP authoring tools report stable `MDV.*` rule IDs. Treat errors as compile blockers; warnings and notes identify behavior that is valid but probably unintended. Fix the graph and compile again rather than suppressing a rule.

## MDV.NullGraph

The validator received no graph object. Reopen or recreate the dialogue asset and make sure custom editor code passes a valid `UMayDialogueGraph` to validation.

## MDV.EmptyGraph

The graph contains no nodes. Add an Entry and Exit, then connect the playable flow between them.

## MDV.MissingEntry

The graph has no Entry node. Every dialogue asset has exactly one entry point; add or restore the Entry node.

## MDV.MissingExit

The graph has no Exit node, so an instance may never complete. Add an Exit and connect every intended terminal path to it.

## MDV.MultipleEntries

The graph contains more than one Entry node. MayDialogue deliberately supports one entry point per asset; keep one Entry and use Branch, Link, or SubGraph nodes for alternate flows.

## MDV.UnconnectedPins

A node has a required output pin with no connection. Connect it to the next node or to Exit. For Player Choice, wire every choice that the player can select.

## MDV.EmptyText

A Say Line or Player Choice lacks usable content, a choice is empty or unwired, or a timeout choice index is invalid. Add localized text or a voice asset where supported, provide at least one valid choice, wire each choice, and keep the timeout index inside the Choices array.

## MDV.UnreachableNodes

A node cannot be reached from Entry. Connect it to the intended flow or delete it if it is obsolete; unreachable nodes never execute.

## MDV.MissingSpeakers

A speaker-dependent node has no speaker or target tag, or its tag is absent from the asset's Speakers list. Select a declared speaker tag and add that speaker to the asset when needed.

## MDV.SelfLoop

A node output is connected directly back to the same node. Route the loop through an explicit state-changing path, or remove the connection if it was accidental.

## MDV.Deadlock

Entry or another reachable node has no path to any Exit. Wire a completion path and ensure cycles have a condition that can leave the cycle.

## MDV.CyclicSubgraph

A cycle in the current graph has no reachable exit. Add an exit branch or restructure the loop so the dialogue can eventually complete.

## MDV.CrossGraphCycle

SubGraph references form a cycle across graph boundaries. Remove one recursive reference or replace recursion with an explicit loop in a single graph that has a reachable exit.

## MDV.Subgraph

A SubGraph node has no assigned graph, has no valid entry identity, references itself, or its target lacks Entry or Exit. Assign a valid subgraph and repair its boundary nodes before compiling the parent.

## MDV.InvalidAssetReference

A Link node has no target dialogue, or it creates an unsupported self-reference. Assign another valid dialogue asset; use local graph flow for loops instead of linking an asset back to itself.

## MDV.MissingRequirements

A Branch has no condition and therefore always takes the True path. Add at least one Requirement, or replace the Branch with a direct connection when no condition is needed.

## MDV.VariableTypeMismatch

The same variable name is written with different MayDialogue types. Choose one of Bool, Int, Float, String, or Tag and make every Set Variable use that type and scope consistently.

## MDV.AsyncWithoutContinuation

An asynchronous node such as Wait, Play Animation, or Camera Focus has no output. Connect its continuation so the instance knows where to resume after the async operation completes.

## MDV.BranchPins

A required True, False, or Default branch output is not connected. Wire every output the node can select, or adjust the branch configuration so the unused path is not possible.

## MDV.WaitNoOp

A Wait node is configured with no meaningful duration or event wait. Set a positive duration or configure the supported external continuation; otherwise remove the node.

## MDV.RandomWeights

A Random Line has no outputs or its weights do not match the output count. Wire at least one output and provide exactly one valid weight per output.

## MDV.CultureKey

A `VoicePerCulture` key is not a recognized culture name. Use an Unreal/IETF culture such as `en`, `en-US`, `de`, or `de-DE`, and keep language-family fallbacks intentional.

## MDV.CameraFocus

Multiple Camera Focus nodes attempt incompatible field-of-view ownership. Ensure only the intended node controls FOV on a path, or disable the conflicting FOV override.

## MDV.ActionFields

An Action node is missing a required asset, class, gameplay tag, variable name, or participant tag. Fill the field named by the diagnostic; spatial actions also need a valid target speaker/participant.

## MDV.SubNodeFields

A Requirement or SideEffect is missing a required variable, tag, class, asset, or participant target. Expand the owning node's sub-node array and complete the field named by the diagnostic.

## MDV.UnlocalizedText

User-facing text was created as culture-invariant text and cannot participate in localization. Re-enter it as localizable `FText`, then export the localization CSV again.
