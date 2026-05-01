---
description: Dialog pausiert und wartet auf ein externes GameplayEvent – perfekt für synchronisierte Cutscene-Momente.
---

# Wait auf externes GameplayEvent

## Szenario

Ein NPC sagt *„Warte, ich zeige dir was."* – dann öffnet sich eine Tür (externes System, Blueprint) und erst danach redet der NPC weiter. Der **Wait**-Node pausiert den Dialog-Flow, bis ein `GameplayTag`-Event eintrifft. Das externe System feuert dieses Event wenn die Tür offen ist.

## Was du lernst

- Wait-Node für `EventTag`-basiertes Warten konfigurieren.
- GameplayEvent von außen in den Dialog feuern (Blueprint + C++).
- Kombination mit Duration-Fallback (maximale Wartezeit).
- Timeout-Verhalten wenn das Event nie kommt.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- Externes System (Tür-Blueprint) kann GameplayEvents feuern.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: NPC – "Warte, ich zeige dir was. Die Tür dort..."]
   │
   ▼
[Wait: EventTag=Interaction.DoorOpened  MaxDuration=10s  TimeoutBehavior=Continue]
   │
   ▼
[SayLine: NPC – "Siehst du? Dahinter liegt das Geheimnis."]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `wait-for-event-graph-overview.png` — Dialog-Graph mit Wait-Node zwischen zwei SayLines.
> *Setup:* Asset `DA_NPC_ShowSecret` geöffnet. SayLine → Wait-Node (Uhr-Icon in Title, grau) → SayLine → Exit. Wait-Node im Details-Panel: `EventTag = Interaction.DoorOpened`, `MaxDuration = 10.0`, `TimeoutBehavior = Continue`.

## Schritt-für-Schritt

### 1. Event-Tag definieren

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Interaction.DoorOpened")
```

### 2. Wait-Node einfügen

Asset: `DA_NPC_ShowSecret`. Nach der ersten SayLine → **Create Node → Wait**:

| Property | Wert |
|----------|------|
| `bWaitForEvent` | `true` |
| `EventTag` | `Interaction.DoorOpened` |
| `bWaitForDuration` | `false` |
| `MaxDuration` | `10.0` |
| `TimeoutBehavior` | `Continue` |

`TimeoutBehavior = Continue` → nach MaxDuration läuft der Dialog weiter, auch wenn das Event nicht kam.

> 📸 **Bild-Platzhalter:** `wait-for-event-node-details.png` — Details-Panel des Wait-Nodes.
> *Setup:* Wait-Node ausgewählt. Details: `bWaitForEvent = true`, `EventTag = Interaction.DoorOpened`, `MaxDuration = 10.0`, `TimeoutBehavior = Continue`. `bWaitForDuration = false`.

### 3. Dialog-Flow verdrahten

Wait-Output → SayLine *„Siehst du? Dahinter liegt das Geheimnis."* → Exit.

### 4. Externes System feuert das Event

**Blueprint – Tür-Actor:**

```text
[Event On Door Fully Opened]
   │
   ▼
[MayDialogueSubsystem → Fire Dialogue Event]
   ├─ EventTag: Interaction.DoorOpened
   └─ Context:  (optional, Tür-Actor)
```

> 📸 **Bild-Platzhalter:** `wait-for-event-bp-door.png` — Blueprint des Tür-Actors mit FireDialogueEvent-Call.
> *Setup:* Tür-Actor Blueprint. `On Door Fully Opened` (Custom Event oder Timeline End) → `Get May Dialogue Subsystem` → `Fire Dialogue Event`. Pins: `EventTag = Interaction.DoorOpened`.

**C++:**

```cpp
// Im Tür-Blueprint oder -Actor, wenn Tür fertig offen ist:
if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
{
    Sub->FireDialogueEvent(TAG_Interaction_DoorOpened, nullptr);
}
```

### 5. Compile und PIE testen

Im PIE: Dialog starten → NPC sagt Zeile → Dialog pausiert am Wait. Tür öffnen → Event feuert → Dialog läuft weiter. MaxDuration ablaufen lassen ohne Event → Dialog läuft nach 10s automatisch weiter.

## Wait-Modi im Überblick

| Modus | Konfiguration | Verhalten |
|-------|---------------|-----------|
| Nur Event | `bWaitForEvent = true`, `bWaitForDuration = false` | Wartet unbegrenzt bis Event kommt |
| Nur Duration | `bWaitForEvent = false`, `bWaitForDuration = true` | Wartet N Sekunden, dann weiter |
| Event OR Duration | `bWaitForEvent = true`, `bWaitForDuration = true` | Wer zuerst kommt |
| Event AND Duration | Nicht direkt, aber: Event erst nach Timer valide | Via FireEvent mit Delay |

## Timeout-Behavior-Optionen

| Option | Verhalten bei MaxDuration |
|--------|--------------------------|
| `Continue` | Dialog läuft weiter als ob Event gekommen |
| `Abort` | Dialog bricht ab (ExitStatus = Failed) |
| `LoopBack` | Wait-Node neu starten (Vorsicht: kann Endlosschleife sein) |

## Variation / Weiter gehen

- **Spieler-Aktion als Trigger**: Nicht eine Tür, sondern der Spieler-Klick auf ein Objekt feuert das Event.
- **Dialog feuert eigenes Event**: FireEvent-Node + Wait-Node in Kombination für ping-pong zwischen Dialog und Spielwelt.
- **Cinematic-Sync**: LevelSequence endet → Event feuert → Dialog-Flow fortsetzt → perfekter Übergang.

## Troubleshooting

**Dialog bleibt hängen, Event kommt nie.**
Das externe System feuert `FireDialogueEvent` nicht oder nutzt einen anderen EventTag. Tag-Name exakt vergleichen. MaxDuration als Sicherheitsnetz setzen.

**Event kommt zu früh (vor dem Wait-Node).**
Der Wait-Node ist noch nicht aktiv. Events die kommen bevor der Wait-Node betreten wird, werden ignoriert. Stelle sicher, dass die Tür-Öffnung nach dem Betreten des Wait-Nodes passiert.

**TimeoutBehavior = Abort, Dialog endet aber kein ExitStatus sichtbar.**
`OnDialogueEnded`-Delegate prüfen – ExitStatus ist `Failed` bei Abort. Das Quest-System muss diesen Fall behandeln.
