---
description: Alle Funktionen des MayDialogueSubsystem mit Use-Case und Beispiel.
---

# Subsystem-API

`UMayDialogueSubsystem` ist der zentrale Orchestrator — ein `UWorldSubsystem`, von dem es pro Welt genau eine Instanz gibt. Er startet Dialoge, hält die aktive Instance am Leben und broadcastet globale Events.

## Zugriff

### Blueprint

```text
[Get MayDialogue Subsystem]   ← Suche nach "MayDialogue" in der Node-Palette
```

`UMayDialogueSubsystem` ist `BlueprintType` — du kannst eine Variable vom Typ `MayDialogue Subsystem Object Reference` anlegen, die Referenz darin speichern und typsicher für Delegate-Binding nutzen.

> 📸 **Bild-Platzhalter:** `subsystem-access-bp.png` — BP-Graph mit Subsystem-Zugriff und gecachter Variable.
> *Setup:* Beliebiges Blueprint. `Event BeginPlay` → `Get MayDialogue Subsystem` → lokale Variable `DialogueSubsystem` (Typ: MayDialogue Subsystem Object Reference). Darunter ein zweiter Bereich mit `Is Valid` auf der Variable.

### C++

```cpp
UMayDialogueSubsystem* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

// Oder als statischer Helper (benötigt UMayDialogueLibrary.h):
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

---

## Dialog starten und stoppen

| Funktion | Use-Case |
|---|---|
| `StartDialogue(Asset, Instigator, Target)` | Dialog starten. Liefert die Instance oder `nullptr` bei Fehler. |
| `CanStartDialogue(Asset, Instigator, Target)` | Prüfen ob ein Start klappen würde, ohne ihn auszulösen (z.B. für Interaktions-Prompts). |
| `StopDialogue(Instance)` | Gezielt eine Instance abbrechen. |
| `StopAllDialogues()` | Alle aktiven Dialoge abbrechen (Level-Wechsel, Spielertod). |

### Beispiel: Dialog starten und Ergebnis prüfen

```text
[Get MayDialogue Subsystem] → [Start Dialogue]
  ├─ Asset:      DA_TraderGreeting
  ├─ Instigator: Player Pawn
  └─ Target:     Trader Actor
       │ Return Value (UMayDialogueInstance)
       ▼
[Is Valid]
  ├─ True  → Alles OK
  └─ False → Log Warning
```

```cpp
if (auto* Inst = Sub->StartDialogue(DA_TraderGreeting, Player, Trader))
{
    // Inst ist die laufende Instance
}
```

### Beispiel: Interaktions-Prompt nur zeigen wenn Dialog möglich

```cpp
bool bCanTalk = Sub->CanStartDialogue(DA_TraderGreeting, Player, Trader);
InteractionPromptWidget->SetVisibility(bCanTalk ? ESlateVisibility::Visible : ESlateVisibility::Hidden);
```

---

## Aktiven Dialog abfragen

| Funktion | Rückgabe | Use-Case |
|---|---|---|
| `GetActiveDialogue()` | `UMayDialogueInstance*` oder `nullptr` | Instance holen um direkt Variable zu lesen oder Delegates zu binden. |
| `IsAnyDialogueActive()` | `bool` | Schnell-Check: läuft gerade irgendwas? |

### Beispiel: Overlap-Trigger nur feuern wenn kein Dialog läuft

```text
[Event Begin Overlap]
    │
    ▼
[Is Any Dialogue Active]  (Get MayDialogue Subsystem → Is Any Dialogue Active)
    │ (bool)
    ▼
[Branch]
  ├─ False → [Start Dialogue] ...
  └─ True  → (nichts tun)
```

```cpp
void ATrigger::OnBeginOverlap(AActor* Other)
{
    if (!Sub->IsAnyDialogueActive())
    {
        Sub->StartDialogue(DA_Hint, Other, this);
    }
}
```

---

## Globale Event-Delegates

Die Subsystem-Delegates feuern **für jeden Dialog** — ideal für globale Listener (Audio-Ducking, Analytics, Quest-Log).

| Delegate | Wann | Parameter |
|---|---|---|
| `OnAnyDialogueStarted` | Direkt nachdem eine neue Instance startet | `UMayDialogueInstance*` |
| `OnAnyDialogueEnded` | Direkt nachdem eine Instance endet (egal ob Completed/Aborted) | `UMayDialogueInstance*` |
| `OnAnyDialogueAborted` | Wenn eine Instance abgebrochen wird — feuert **vor** `OnAnyDialogueEnded` | `UMayDialogueAsset*, EMayDialogueExitStatus, float, AActor*, AActor*` |

### `FireDialogueEvent` — Event per Tag auslösen

Feuert ein benanntes Dialogue-Event in der aktuell laufenden Instance (entspricht einem FireEvent-Node im Graphen, aber aus externem Code):

```text
[Get MayDialogue Subsystem]
    │
    ▼
[Fire Dialogue Event]
  ├─ World Context: Self
  └─ Event Tag:     Dialogue.Event.LightsOut
```

```cpp
UMayDialogueLibrary::FireDialogueEvent(this, TAG_Dialogue_Event_LightsOut);
// oder direkt am Subsystem:
Sub->FireDialogueEvent(this, TAG_Dialogue_Event_LightsOut);
```

### Participant-Lookups

| Funktion | Art | Rückgabe |
|---|---|---|
| `Get All Participants` | Pure | `TArray<UMayDialogueParticipant*>` aller aktiven Participants in der Welt |
| `Find Participant By Tag` | Pure | `UMayDialogueParticipant*` für einen bestimmten `FGameplayTag`, oder `nullptr` |

Beide sind in der `UMayDialogueLibrary` als `BlueprintPure` verfügbar (kein Subsystem-Zugriff nötig).

---

### Blueprint: Event binden

> 📸 **Bild-Platzhalter:** `subsystem-delegate-bind-bp.png` — BP-Graph für Event-Binding am Subsystem.
> *Setup:* Quest-Director-Blueprint, `Event BeginPlay`. `Get MayDialogue Subsystem` → `Bind Event to On Any Dialogue Ended` → Custom Event `Handle Dialogue Ended` (Input-Pin: `Instance` vom Typ `MayDialogueInstance Object Reference`). Custom Event hat eine `Branch`-Node darunter: `Get Dialogue Asset` von der Instance verglichen mit einer Asset-Referenz.

```text
[Event BeginPlay]
    │
    ▼
[Get MayDialogue Subsystem]
    │
    ├─→ [Bind Event to On Any Dialogue Started]  ──► Custom Event: On Dialogue Started
    └─→ [Bind Event to On Any Dialogue Ended]    ──► Custom Event: On Dialogue Ended
```

### C++: Audio-Ducking-Beispiel

```cpp
void UAudioManager::BeginPlay()
{
    if (auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>())
    {
        Sub->OnAnyDialogueStarted.AddDynamic(this, &UAudioManager::DuckMusic);
        Sub->OnAnyDialogueEnded.AddDynamic(this, &UAudioManager::RestoreMusic);
    }
}

void UAudioManager::DuckMusic(UMayDialogueInstance* Instance)
{
    MusicComponent->SetVolumeMultiplier(0.2f);
}

void UAudioManager::RestoreMusic(UMayDialogueInstance* Instance)
{
    MusicComponent->SetVolumeMultiplier(1.0f);
}
```

---

## Read/Write direkt am Subsystem

Das Subsystem stellt alle Lese- und Schreib-Methoden der aktiven Instance als Blueprint-Callable bereit — du brauchst dafür keine Instance-Referenz zu halten.

Vollständige Beschreibung: [Read/Write-API](read-write-api.md).

---

## Automatisches Tick-Management

Das Subsystem läuft automatisch jeden Frame und kümmert sich um:

- Auto-Advance-Timer (AdvanceMode `Timer`)
- Choice-Timeouts
- Camera-Blend-Fortschritt
- Cleanup beendeter Instances am Frame-Ende

Du musst nichts selbst ticken.

{% hint style="info" %}
Beim Level-Travel wird das Subsystem zerstört. Es ruft dabei automatisch `StopAllDialogues()` auf — alle offenen Dialoge werden sauber beendet.
{% endhint %}
