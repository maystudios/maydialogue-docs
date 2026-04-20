# Participants & Sprecher

Jeder Actor, der an einem Dialog teilnimmt, braucht eine **`UMayDialogueParticipant`**-Komponente. Jeder Sprecher, der in einem Dialog-Asset vorkommt, hat eine **`FMayDialogueSpeaker`**-Definition. Dieses Kapitel zeigt, wie die beiden zusammenspielen.

## Participant vs. Speaker

| | Participant (Actor-Seite) | Speaker (Asset-Seite) |
| --- | --- | --- |
| Lebt als | `UActorComponent` auf einem Actor | `FMayDialogueSpeaker`-Struct im Dialog-Asset |
| Zweck | *„Wer ist das im Level?"* | *„Wie wird er in **diesem** Dialog präsentiert?"* |
| Felder | Tag, DisplayName, PersistentMemory, AudioOverrides, Kamera-Offsets | DisplayName, Portrait, NodeColor, Audio-Tuning, Babel-Profil |
| Beziehung | 1:1 Tag-Matching | Wird pro Asset neu definiert |

Warum diese Trennung? Weil ein NPC in verschiedenen Dialogen anders präsentiert werden darf. Derselbe Wächter-Actor (Participant) kann im ersten Dialog als *„Fremder Wächter"* mit grauem Portrait erscheinen und später als *„Hans, Hauptmann"* mit persönlichem Portrait – jedes Asset definiert seine eigenen Speaker-Strukturen, die sich per Tag an die Actor-Participants binden.

## Die Participant-Komponente

`UMayDialogueParticipant` (kurz: **UMDP**) ist ein `UActorComponent`, den du an jeden Dialog-relevanten Actor hängst.

### Wichtige Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `ParticipantTag` | `FGameplayTag` | **Primärer Match-Key**. Dialoge identifizieren Teilnehmer hierüber. Convention: `Dialogue.Speaker.<Name>`. |
| `DisplayName` | `FText` | Fallback-Name. Wird vom Speaker im Asset überschrieben. |
| `Portrait` | `TSoftObjectPtr<UTexture2D>` | Fallback-Portrait. Wird vom Speaker im Asset überschrieben. |
| `ParticipantTags` | `FGameplayTagContainer` | Zusätzliche Tags (z.B. `Dialogue.Trait.Hostile`) für Branching. |
| `DefaultDialogue` | `UMayDialogueAsset*` | Das Asset, das bei Standard-Interaktion startet. |
| `AttenuationOverride` | `USoundAttenuation*` | Voice-Räumlichkeit, die diesem Actor eigen ist. |
| `bAutoFacePartner` | bool | Drehe den Actor automatisch zum Gesprächspartner. |
| `CameraTargetOffset` | `FVector` | Offset vom Actor-Origin für CameraFocus-Nodes. |
| `PersistentMemory` | `FInstancedPropertyBag` (SaveGame) | Variablen, die den Participant überdauern. |

### Die drei Start-Pfade

Ein Dialog kann auf drei Wegen angestoßen werden; alle enden beim Subsystem.

#### 1. Auf der Komponente

```cpp
// In einem Interaction-Trigger am Guard-Actor
UMayDialogueParticipant* Guard = GuardActor->FindComponentByClass<UMayDialogueParticipant>();
UMayDialogueParticipant* Player = PlayerPawn->FindComponentByClass<UMayDialogueParticipant>();

Guard->StartDefaultDialogue(Player);
```

Vorteil: OOP-Stil, lebt direkt am NPC.

#### 2. Über die Library

```cpp
UMayDialogueLibrary::StartDialogue(
    this,                    // WorldContext
    DA_Greeting_Simple,      // Asset
    PlayerPawn,              // Instigator
    GuardActor               // Target
);
```

Vorteil: Blueprint-nah, funktionaler Stil.

#### 3. Über das Subsystem

```cpp
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(this);
Sub->StartDialogue(DA_Greeting_Simple, PlayerPawn, GuardActor);
```

Vorteil: Zentralistisch, praktisch für System-Code (z.B. Quest-Script).

Alle drei erzeugen am Ende eine `UMayDialogueInstance`.

### Dynamisches Dialog-Swapping

`SetActiveDialogue(NewAsset)` auf der Komponente überschreibt den Default. `GetActiveDialogue()` liefert den effektiven Dialog (Override wenn gesetzt, sonst Default).

Typisch: Quest-System markiert „NPC hat jetzt ein anderes Gespräch", z.B. nach einem Event:

```cpp
Guard->SetActiveDialogue(DA_Guard_AfterBetrayal);
```

## Sprecher im Asset

Die Sprecher-Liste im Asset-Editor hat für jeden Eintrag:

| Property | Zweck |
| --- | --- |
| `SpeakerTag` | Match-Key zum Participant-Tag. |
| `DisplayName` | Der Name, der im UI erscheint. |
| `Portrait` | Das Portrait, das im UI erscheint. |
| `NodeColor` | Title-Bar-Farbe der SayLine-Nodes dieses Sprechers. |
| `AudioModeOverride` | Default / Spatial3D / Force2D. |
| `SoundClassOverride` | Eigene SoundClass für diesen Sprecher. |
| `AttenuationOverride` | Eigene Attenuation. |
| `VolumeMultiplier` / `PitchMultiplier` | Tonhöhen-/Lautstärke-Tuning. |
| `BabelProfile` | Eigenes Babel-Profile (wenn keine echte Stimme). |

## Participant-Resolution zur Laufzeit

Wenn ein SayLine-Node den Speaker-Tag `Dialogue.Speaker.Guard` ansteuert, sucht die Instance nach einer passenden `UMayDialogueParticipant` mit identischem Tag **unter den am Dialog beteiligten Actors**:

1. Zunächst Instigator-Actor.
2. Dann Target-Actor.
3. Danach alle dynamisch hinzugefügten Participants (z.B. zusätzliche NPCs, die mitsprechen).
4. Wenn kein Match: Fallback auf eine reine Asset-Präsentation (Portrait + DisplayName kommen aus dem Speaker), und audio-seitig 2D-Fallback.

## PersistentMemory

Die **persistente Seite** des Participants:

```cpp
// Setter
Guard->SetPersistentBool("HasMet", true);
Guard->SetPersistentInt("FriendshipPoints", 42);
Guard->SetPersistentTag("LastMood", FGameplayTag("Dialogue.Mood.Friendly"));

// Getter mit Default
bool HasMet = Guard->GetPersistentBool("HasMet", false);
int32 Points = Guard->GetPersistentInt("FriendshipPoints", 0);
```

Gespeichert wird im `FInstancedPropertyBag`, markiert als `SaveGame`. Ein Projekt-eigenes SaveGame-System packt das automatisch ein. Für schnellen Einstieg gibt es den [QuickSave-Helper](../persistence/quicksave-helper.md).

## Netz-Awareness (Multiplayer-Ready)

Die Komponente ist **repliziert** und hat Server-/Client-RPC-Pfade:

| Richtung | RPC |
| --- | --- |
| Client → Server | `ServerAdvanceConversation()`, `ServerSelectChoice(Index)` |
| Server → Client | `ClientStartConversation()`, `ClientExitConversation()`, `ClientUpdateConversation(...)` |
| Replicated | `CurrentMessage` (OnRep), `CurrentChoices` (OnRep), `ConversationsActive` (OnRep) |

{% hint style="warning" %}
**Phase 2 ist Work-in-Progress.** `ClientUpdateConversation` und die Net-Serialisierung von `FMayDialogueMessage` / `FMayDialogueChoiceEntry` sind als TODO markiert. Singleplayer funktioniert komplett; Multiplayer-Setups brauchen zusätzliche Arbeit.
{% endhint %}

## Zusammengefasst

* **Participants** sind die Actor-seitige Identität (Tag, Persistent-Memory, Audio-Overrides, Kamera-Offsets).
* **Sprecher** sind die Asset-seitige Präsentation (DisplayName, Portrait, NodeColor, Audio-Tuning).
* Die Bindung erfolgt über **Tag-Matching**.
* `DefaultDialogue` startet am NPC, `SetActiveDialogue` ermöglicht dynamisches Swapping.
* PersistentMemory ist der Ort für Variablen, die einen Dialog überdauern.

Weiter: [Variablen & Scopes](variables-scopes.md).
