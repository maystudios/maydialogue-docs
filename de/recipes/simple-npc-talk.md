---
description: Minimales NPC-Gespräch mit zwei SayLines – der perfekte Einstieg in MayDialogue.
---

# Einfaches NPC-Gespräch

## Szenario

Ein Dorfbewohner begrüßt den Spieler, sagt zwei Sätze und das Gespräch endet. Kein Branching, keine Bedingungen, kein GAS. Dieses Rezept schärft dein Gefühl dafür, wie `Entry`, `SayLine` und `Exit` zusammenspielen – und welcher `AdvanceMode` wann sinnvoll ist.

## Was du lernst

- Ein Dialog-Asset anlegen und benennen.
- Speakers-Panel befüllen (Tag + DisplayName).
- SayLine-Nodes verbinden und `AdvanceMode` setzen.
- Den Dialog im Preview-Runner testen, ohne PIE zu starten.

## Voraussetzungen

- [Quick Start](../getting-started/quick-start.md) abgeschlossen.
- Plugin aktiviert, Widget in Projekt-Einstellungen konfiguriert.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Dorfbewohner – "Willkommen in Kreuzhof, Fremder."  AdvanceMode: Manual]
   │
   ▼
[SayLine: Dorfbewohner – "Nachts gehen komische Dinge um. Pass auf dich auf."  AdvanceMode: Manual]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `simple-npc-talk-graph-overview.png` — Übersichtsansicht des fertigen Graphen im Asset-Editor.
> *Setup:* Asset `DA_Villager_Intro` geöffnet. Von links nach rechts: grüner `Entry`-Node → `SayLine "Willkommen in Kreuzhof"` (Title-Bar warm-gelb, Sprecher: Dorfbewohner) → `SayLine "Nachts gehen..."` (gleiche Farbe) → roter `Exit: Completed`-Node. Verbindungslinien gut sichtbar, kein Comment-Block.

## Schritt-für-Schritt

### 1. Asset anlegen

Im Content Browser: **Rechtsklick → Miscellaneous → May Dialogue Asset**. Name: `DA_Villager_Intro`.

### 2. Speaker registrieren

**Window → Speakers** öffnen. Klick auf **Add Speaker**:

| Feld | Wert |
|------|------|
| `DisplayName` | Dorfbewohner |
| `SpeakerTag` | `Dialogue.Speaker.Villager` |
| `Color` | Warm-Gelb (Picker) |

> 📸 **Bild-Platzhalter:** `simple-npc-talk-speakers-panel.png` — Speakers-Panel mit ausgefülltem Eintrag.
> *Setup:* Speakers-Panel geöffnet (Tab links). Einziger Eintrag: `DisplayName = "Dorfbewohner"`, `SpeakerTag = Dialogue.Speaker.Villager`, Farb-Chip warm-gelb.

### 3. Nodes verbinden

1. Vom **Entry**-Output-Pin einen Draht ziehen → **Create Node → Say Line**.
2. `SpeakerTag` = `Dialogue.Speaker.Villager`, `DialogueText` = *„Willkommen in Kreuzhof, Fremder."*, `AdvanceModeOverride` = `Manual`.
3. Vom Output dieser SayLine eine zweite SayLine anlegen: *„Nachts gehen komische Dinge um. Pass auf dich auf."*
4. Vom Output der zweiten SayLine → **Exit**-Node, `ExitStatus = Completed`.
5. **Compile** (Toolbar oben).

> 📸 **Bild-Platzhalter:** `simple-npc-talk-sayline-details.png` — Details-Panel der ersten SayLine.
> *Setup:* Erste SayLine im Graph ausgewählt. Details-Panel zeigt: `SpeakerTag = Dialogue.Speaker.Villager`, `DialogueText = "Willkommen in Kreuzhof, Fremder."`, `AdvanceModeOverride = Manual`. Voice-Slot leer.

### 4. Im Preview-Runner testen

Klick auf **Play** (Toolbar) → der Dialog läuft direkt im Editor. Advance mit **Space**, Abbruch mit **Esc**.

## Advance-Modi im Überblick

| Modus | Advance durch |
|-------|---------------|
| `Manual` | Spieler-Input (Klick / Enter) |
| `Timer` | `AutoAdvanceDelay` verstreicht |
| `AfterVoice` | Voice-Asset-Wiedergabe endet |
| `Immediate` | Sofort, ohne Warten |
| `AfterAnimation` | Montage auf Participant endet |

Für interaktive NPC-Gespräche: **Manual**. Für Cutscenes ohne Spieler-Input: **Timer** oder **AfterVoice**.

## Blueprint-Triggering

Wenn der Spieler den NPC interagiert (z.B. über eine Interaction-Komponente):

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Villager_Intro
   ├─ Instigator: Player Pawn
   └─ Target:     Self (der NPC-Actor)
```

> 📸 **Bild-Platzhalter:** `simple-npc-talk-bp-trigger.png` — Blueprint-Graph des Trigger-Events.
> *Setup:* Beliebiger Actor-Blueprint geöffnet. Event-Graph zeigt: `Event OnInteract` → `Start Dialogue (MayDialogueLibrary)` mit gefüllten Pins: `Asset = DA_Villager_Intro`, `Instigator = Get Player Pawn`, `Target = Self`.

{% hint style="info" %}
**C++-Variante**

```cpp
if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
{
    Sub->StartDialogue(DA_Villager_Intro, PC->GetPawn(), this);
}
```
{% endhint %}

## Variation / Weiter gehen

- Ersetze eine SayLine durch eine **RandomLine** für variable Begrüßungen → [Zufällige Begrüßungen](random-greetings.md).
- Füge einen **PlayerChoice**-Node hinzu, damit der Spieler antworten kann → [Verzweigungen mit Bedingungen](branching-conditions.md).
- Lagere die SayLines als eigenes Fragment aus → [Wiederverwendbare Dialog-Fragmente](linking-dialogues.md).

## Troubleshooting

**Dialog endet sofort nach der ersten Zeile.**
Prüfe: Output-Pin von SayLine 1 zeigt direkt auf `Exit` statt auf SayLine 2. Oder `AdvanceModeOverride = Immediate` gesetzt – wechsle auf `Manual`.

**Spieler-Input löst Advance nicht aus.**
In den [Projekt-Einstellungen](../getting-started/project-settings.md) muss `bSwitchToUIInputDuringDialogue = true` sein. Außerdem muss das Widget Focus bekommen (`SetKeyboardFocus` in `NativeConstruct`).

**Name "Dorfbewohner" erscheint leer im Widget.**
Im Speakers-Panel ist nur der Tag gesetzt, aber kein `DisplayName`. Trage den Anzeigenamen im Panel ein.
