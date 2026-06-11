---
description: Branch-Node und GAS-Requirement kombinieren – Dialog reagiert auf den Spieler-Zustand.
---

# Verzweigungen mit Bedingungen

## Szenario

Ein Wachmann am Stadttor reagiert unterschiedlich, je nachdem ob der Spieler den Tag `Story.Pass.CityGate` trägt. Mit Pass kommt er durch, ohne Pass wird er abgewiesen und bekommt eine PlayerChoice zur Lösung. Das ist das Standard-Muster für Weltreaktivität in MayDialogue.

## Was du lernst

- Branch-Node anlegen und seine `Condition` konfigurieren.
- HasTag-Requirement als Branch-Condition konfigurieren.
- Die True- / False-Outputs (und den optionalen Default-Fallback) verdrahten.
- Den Unterschied zwischen Branch (ganzer Pfad) und Choice-Requirement (einzelne Option).

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- GAS im Projekt aktiv, PlayerController hat einen ASC.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: HasTag(Story.Pass.CityGate) → [SayLine: "Willkommen zurück."] → [Exit: Completed]
   └─ BP2: <Fallback>                  → [SayLine: "Halt! Wer geht da?"] → [PlayerChoice]
                                                                               ├─ "Ich habe keinen Pass." → [Exit: Failed]
                                                                               └─ "Ich suche den Torvorsteher." → [SayLine: "Kammer links."] → [Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `branching-conditions-graph-overview.png` — Übersichtsgraph des Wachmann-Dialogs.
> *Setup:* Asset `DA_Guard_Gate` geöffnet. Sichtbar: `Entry` → `Branch`-Node (Diamant-Form, zwei Ausgangs-Pins). Oberer Pin führt zu `SayLine "Willkommen zurück"` → `Exit`. Unterer Pin führt zu `SayLine "Halt!"` → `PlayerChoice` mit zwei Choice-Sub-Nodes → zwei Ausgänge zu je einem Exit. Layout horizontal, kein Comment-Block.

## Schritt-für-Schritt

### 1. Asset und Speaker anlegen

Asset: `DA_Guard_Gate`. Speaker im Speakers-Panel: `DisplayName = Wachmann`, `SpeakerTag = Dialogue.Speaker.Guard`.

### 2. Branch-Node einfügen

Vom Entry-Output → **Create Node → Branch**. Der Branch-Node zeigt im Graph einen Input und zwei Ausgangs-Pins, **True** und **False**.

### 3. Condition konfigurieren

Branch-Node auswählen. Im Details-Panel die `Condition` auf eine **HasTag**-Requirement setzen:

| Property | Wert |
|----------|------|
| `RequiredTag` | `Story.Pass.CityGate` |
| `bCheckOnInstigator` | `true` (prüft den Spieler, nicht den NPC) |

Passt die Condition, nimmt der Branch seinen **True**-Output; schlägt sie fehl (`FailedButVisible` oder `FailedAndHidden`, beide als „false" behandelt), den **False**-Output. Lässt du die `Condition` leer, nimmt der Branch immer True.

> 📸 **Bild-Platzhalter:** `branching-conditions-branch-details.png` — Details-Panel des Branch-Nodes mit HasTag-Requirement.
> *Setup:* Branch-Node ausgewählt. Details-Panel zeigt die `Condition` als HasTag-Requirement mit `RequiredTag = Story.Pass.CityGate`, `bCheckOnInstigator = true` und `bHasFallback = false`.

### 4. Erfolgs-Pfad verdrahten

Branch **True**-Output-Pin → SayLine *„Willkommen zurück. Tor ist offen."* → Exit (`Completed`).

### 5. Ablehnungs-Pfad verdrahten

Branch **False**-Output-Pin → SayLine *„Halt! Wer geht da?"* → PlayerChoice mit zwei Choices:
- Choice 1: *„Ich habe keinen Pass."* → Exit (`Failed`).
- Choice 2: *„Ich suche den Torvorsteher."* → SayLine *„Dann geh zur Kammer links."* → Exit (`Completed`).

### 6. Compile und testen

Im Preview-Runner läuft der Dialog mit dem False- (Ablehnungs-)Pfad, solange kein Tag gesetzt ist. Füge im Debugger testweise den Tag am Player-ASC hinzu und prüfe den True- (Erfolgs-)Pfad.

## Branch-Auswertungslogik

Der Branch wertet seine einzelne `Condition`-Requirement aus und routet entsprechend:

| Condition-Ergebnis | Genommener Output |
|----------|-----------|
| `Passed` | **True**-Output. |
| `FailedButVisible` | **False**-Output. |
| `FailedAndHidden` | **False**-Output. |

Aktiviere `bHasFallback` für einen dritten **Default**-Output bei Degenerate-Fällen (z.B. der Instigator hat kein ASC, sodass die Tag-Prüfung gar nicht laufen kann). Für mehr als ein True/False-Split (z.B. completed / active / not-started) **verkette Branch-Nodes** — leite den False-Output eines Branch in den nächsten Branch.

## GAS-Tag für den Test setzen

```cpp
// Irgendwo im Quest-System, wenn der Schritt abgeschlossen wird:
UAbilitySystemComponent* ASC = UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(PlayerPawn);
ASC->AddLooseGameplayTag(FGameplayTag::RequestGameplayTag(TEXT("Story.Pass.CityGate")));
```

{% hint style="info" %}
Für **persistente** Tags (die ein Speicherstand überleben sollen) nutze einen Infinite-Duration GameplayEffect mit `GrantedTags`. `AddLooseGameplayTag` wird nicht automatisch gespeichert.
{% endhint %}

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Guard_Gate
   ├─ Instigator: Get Player Pawn
   └─ Target:     Self
```

> 📸 **Bild-Platzhalter:** `branching-conditions-bp-trigger.png` — Blueprint-Trigger am NPC.
> *Setup:* Wachmann-Actor BP. Event-Graph: `Event OnInteract` → `Start Dialogue`. Pins: `Asset = DA_Guard_Gate`, `Instigator = Get Player Pawn (0)`, `Target = Self`.

{% hint style="info" %}
**C++-Variante**

```cpp
void AGuardNPC::BeginConversation(AActor* Player)
{
    if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
    {
        Sub->StartDialogue(GuardAsset, Player, this);
    }
}
```
{% endhint %}

## Variation / Weiter gehen

- Requirement an **PlayerChoice** statt an Branch hängen → nur eine einzelne Option verstecken. Siehe [Choice nur sichtbar mit Tag](choice-with-tag-requirement.md).
- Attribut statt Tag prüfen (z.B. Stamina ≥ 50) → [Choice mit Attribut-Bedingung](choice-with-attribute-requirement.md).
- Ablehnungs-Pfad als eigenes Asset auslagern und per Link referenzieren → [Wiederverwendbare Dialog-Fragmente](linking-dialogues.md).

## Troubleshooting

**Dialog nimmt immer den Fallback-Pfad.**
Öffne den [Debugger](../editor/debugger.md) in PIE und prüfe, ob der Tag wirklich am Spieler-ASC hängt. `Story.Pass.CityGate` ≠ `Story.Pass.City.Gate` – exakte Schreibweise erforderlich. Außerdem: `bCheckOnInstigator` muss `true` sein, sonst wird der NPC-ASC geprüft.

**Erfolgs-Pfad nie erreicht, obwohl Tag vorhanden.**
`bCheckOnInstigator = false` → Requirement prüft den NPC statt den Spieler. Oder der Actor hat keinen ASC. Requirement liefert dann immer `Fail`.

**Branch fällt unerwartet immer auf eine Seite.**
Prüfe die `Condition` erneut: Ein `FailedButVisible`- oder `FailedAndHidden`-Ergebnis routet beides auf den **False**-Output. Brauchst du einen dritten Pfad für den Fall „Condition konnte nicht ausgewertet werden" (z.B. fehlendes ASC), aktiviere `bHasFallback` und verdrahte den Default-Output.
