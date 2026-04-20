# Einen Dialog starten

Drei Wege führen zum Ziel. Alle enden in `UMayDialogueSubsystem::StartDialogue(...)`.

## Variante 1 – Auf der Participant-Komponente

OOP-Stil. Praktisch in Interaction-Triggern am NPC.

### C++

```cpp
void AGuardActor::OnInteract(AActor* Interactor)
{
    auto* GuardPart = FindComponentByClass<UMayDialogueParticipant>();
    auto* PlayerPart = Interactor->FindComponentByClass<UMayDialogueParticipant>();
    if (!GuardPart || !PlayerPart) return;

    GuardPart->StartDefaultDialogue(PlayerPart);
}
```

### Blueprint

```
[Event OnInteract]
  │
  ▼
[Get Component by Class: MayDialogueParticipant] (auf Self)
  │
  ▼
[Start Default Dialogue]
  ├ Other: [Get Component by Class MayDialogueParticipant] (auf Interactor)
```

## Variante 2 – Über die Library

Funktional. Blueprint-freundlich.

### C++

```cpp
UMayDialogueLibrary::StartDialogue(
    this,              // WorldContext
    DA_Greeting,       // Asset
    PlayerPawn,        // Instigator
    GuardActor         // Target
);
```

### Blueprint

```
[MayDialogueLibrary :: Start Dialogue]
  ├ World Context: Self
  ├ Asset:         DA_Greeting
  ├ Instigator:    Player Pawn
  └ Target:        Guard Actor
```

## Variante 3 – Über das Subsystem

Zentralistisch. Nützlich in System-Code (Quest-Script, Cutscene-Regie).

### C++

```cpp
if (auto* Sub = UMayDialogueSubsystem::Get(this))
{
    Sub->StartDialogue(DA_Greeting, PlayerPawn, GuardActor);
}
```

### Blueprint

```
[Get May Dialogue Subsystem] → [Subsystem :: Start Dialogue]
```

## Pre-Flight-Check

Wenn du vorab wissen willst, ob ein Dialog möglich wäre:

```cpp
bool bCanStart = Sub->CanStartDialogue(Asset, Instigator, Target);
```

Prüft:

* Asset ist gültig und hat Entry.
* Instigator & Target haben (wo nötig) Participants mit passenden Tags.
* Kein anderer Dialog blockiert.

## Nur ein Dialog gleichzeitig

Das Plugin **erzwingt, dass maximal ein Dialog pro Welt läuft**. Wenn du einen neuen startest, während ein anderer aktiv ist, wird der alte **abgebrochen** (Cleanup vollständig). Kein Crash, kein Queue.

## Blocking-Pattern für Interaction

Wenn dein Trigger nur feuern soll, solange der Spieler wirklich am NPC steht:

```cpp
void AGuardActor::OnBeginOverlap(AActor* Other)
{
    if (PlayerIsValid(Other) && !Sub->IsAnyDialogueActive())
    {
        StartDialogue();
    }
}
```

## Auto-Dialogue beim NPC-Spawn

Manchmal soll ein NPC direkt beim Erscheinen reden (*„Hallo! Endlich bist du da!"*):

```cpp
void AGuardActor::BeginPlay()
{
    Super::BeginPlay();

    // einige Frames warten, damit Spieler spawned ist
    GetWorld()->GetTimerManager().SetTimer(
        AutoStartTimer,
        this,
        &AGuardActor::TryAutoStart,
        1.0f,
        false
    );
}
```

## Abbrechen von außen

```cpp
Sub->StopAllDialogues();              // harte Cleanup, alle abbrechen
Sub->StopDialogue(ActiveInstance);    // gezielt
```

Der `StopAllDialogues` ist praktisch beim Level-Wechsel oder Spieler-Tod.

## Weiter

* [Subsystem-API](subsystem-api.md) – alle Methoden im Detail.
* [Bridge & Lifecycle-Events](bridge-events.md) – wie externe Systeme hinhören.
