# Glossar

Kurzes Begriffs-Verzeichnis der häufigsten MayDialogue-Termini.

| Begriff | Bedeutung |
| --- | --- |
| **Asset** | `UMayDialogueAsset` – das Blueprint-Asset, das Nodes, Speakers und Variables enthält. |
| **Instance** | `UMayDialogueInstance` – das *laufende* Objekt eines Gesprächs. Erzeugt beim Start, zerstört beim Ende. |
| **Subsystem** | `UMayDialogueSubsystem` – zentrale Welt-Autorität für Dialog-Orchestrierung. |
| **Participant** | `UMayDialogueParticipant` – ActorComponent, macht einen Actor zum Dialog-Teilnehmer. |
| **Speaker** | `FMayDialogueSpeaker` – Asset-seitige Definition eines Sprechers (Name, Portrait, Farbe, Audio). |
| **Node** | Ein Element im Dialog-Graph. Leitet von `UMayDialogueNode_Base` ab. |
| **Sub-Node** | Requirement, Choice oder SideEffect – lebt als Pill im Body eines Eltern-Nodes. |
| **Action-Node** | Node, der eine Aktion ausführt (CameraFocus, ApplyEffect, …). Prominente Box im Graph. |
| **Core-Node** | Struktureller Node (Entry, Exit, SayLine, PlayerChoice, Branch, …). |
| **TaskResult** | `FMayDialogueTaskResult` – Rückgabetyp von `ExecuteNode`. Steuert den Flow-Pointer. |
| **Context** | `FMayDialogueContext` – Info-Struct, das an jeden Node-Execute gereicht wird (Instance, Instigator, Target, ASC). |
| **Scope-Stack** | Return-Mechanismus für Link-/SubGraph-Sprünge. |
| **Advance-Mode** | Wie kommt der nächste Node dran? Manual, Timer, AfterVoice, AfterAnimation, Immediate. |
| **Fail-Behavior** | Was passiert, wenn Node-Requirements scheitern? Skip oder Abort. |
| **Dialogue-Scope** | Variablen, die nur während eines Gesprächs leben. |
| **Participant-Scope** | Variablen, die einen Participant überdauern, optional per SaveGame persistiert. |
| **Emotion-Tags** | `FGameplayTagContainer` auf SayLines – hierarchische Meta-Tags für UI/Audio/Animation. |
| **Choice-Tags** | `FGameplayTagContainer` auf Choices – Meta für externe Systeme. |
| **Babel** | Prozedurale Nonsense-Stimmen-Engine. Blip- oder Phoneme-Mode. |
| **Bridge** | `IMayDialogueBridge` – generisches Interface für externe Systeme. |
| **Library** | `UMayDialogueLibrary` – Blueprint-freundliche Wrapper-Functions. |
| **Validator** | `FMayDialogueValidator` – Compile-Zeit-Sanity-Checker im Editor. |
| **Compiler** | `FMayDialogueCompiler` – wandelt UEdGraph in Runtime-Node-Map um. |
| **Debugger** | `FMayDialogueDebugger` – PIE-seitiges Breakpoint-System. |
| **Preview Runner** | `FMayDialoguePreviewRunner` – Play-Dialog-ohne-PIE im Asset-Editor. |
| **Outline** | Flache Node-Liste mit Suche + Filter im Editor. |
| **Knot** | Editor-Reroute-Node ohne Runtime-Repräsentation. |
| **SubGraph** | Eingebetteter Dialog-Teilgraph innerhalb desselben Assets. |
| **Link-Node** | Sprung in anderes Dialog-Asset mit optionalem Return. |
| **ASC** | `UAbilitySystemComponent` – GAS-Herzstück am Actor. |
| **GE** | `UGameplayEffect` – Wirkeinheit in GAS. |
| **LooseTag** | Per `AddLooseGameplayTag` gesetzter Tag am ASC (nicht GE-granted). |
