---
description: PlayerChoice mit Countdown-Timer – bei Ablauf wird automatisch eine Default-Choice gewählt.
---

# Timed Choice (Auto-Auswahl)

## Szenario

Ein Gegner gibt dem Spieler 10 Sekunden, um sich zu ergeben. Reagiert er nicht, wählt das System automatisch *„Ich ergebe mich nicht"* und der Kampf beginnt. Dieses Muster erzeugt echten Zeitdruck ohne Cutscene. Der Countdown-Timer läuft direkt im Dialog-Widget.

## Was du lernst

- Choice-Timeout am PlayerChoice-Node konfigurieren.
- Default-Choice für Auto-Select festlegen.
- Das Widget zeigt den Countdown automatisch an.
- Kombination mit Sound-Stinger für Dringlichkeit.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Gegner – "Du hast 10 Sekunden. Ergib dich oder stirb."]
   │
   ▼
[PlayerChoice  Timeout: 10s  DefaultChoice: 1]
   ├─ [0] "Ich ergebe mich."    → [SayLine: "Klug."] → [Exit: Completed]
   └─ [1] "Niemals!"  (Default) → [SayLine: "Dann stirb."] → [FireEvent: Combat.Start] → [Exit: Failed]
```

> 📸 **Bild-Platzhalter:** `timed-choice-graph-overview.png` — PlayerChoice-Node mit Timeout-Einstellung.
> *Setup:* Asset `DA_Enemy_Ultimatum` geöffnet. SayLine → PlayerChoice mit zwei Choices. Im Details-Panel des PlayerChoice: `Timeout = 10.0`, `DefaultChoiceIndex = 1` (Index des "Niemals!"-Choice) sichtbar.

## Schritt-für-Schritt

### 1. PlayerChoice-Node anlegen

Asset: `DA_Enemy_Ultimatum`. Speaker: `Dialogue.Speaker.Enemy`. SayLine *„Du hast 10 Sekunden..."* → **PlayerChoice**-Node.

### 2. Choices befüllen

Choice 0 (Index 0): `ChoiceText = "Ich ergebe mich."`.
Choice 1 (Index 1): `ChoiceText = "Niemals!"`.

### 3. Timeout konfigurieren

Am PlayerChoice-Node im Details-Panel:

| Property | Wert |
|----------|------|
| `bEnableTimeout` | `true` |
| `TimeoutDuration` | `10.0` |
| `DefaultChoiceIndex` | `1` (Index der "Niemals!"-Choice) |

`DefaultChoiceIndex = 1` → wenn der Timer abläuft, wird Choice 1 automatisch ausgewählt.

> 📸 **Bild-Platzhalter:** `timed-choice-player-choice-details.png` — Details-Panel mit Timeout-Settings.
> *Setup:* PlayerChoice-Node ausgewählt. Details: `bEnableTimeout = true`, `TimeoutDuration = 10.0`, `DefaultChoiceIndex = 1`. Darunter die zwei Choices mit ihren Texten.

### 4. Outputs verdrahten

Choice 0 → SayLine *„Klug. Du lebst einen weiteren Tag."* → Exit (`Completed`).
Choice 1 → SayLine *„Dann stirb."* → **FireEvent**-Node (`Combat.Start`) → Exit (`Failed`).

### 5. Countdown im Widget

Das Standard-Dialog-Widget zeigt den Countdown-Timer automatisch an, sobald `bEnableTimeout = true`. Der Timer erscheint als Fortschrittsbalken oder Zahl über der Choice-List. Bei eigenem UMG-Widget muss du das `CountdownValue`-Binding selbst implementieren (siehe [Eigenes UMG-Widget](custom-umg-widget.md)).

### 6. Compile und PIE testen

Im PIE: Dialog starten. 10 Sekunden warten → Choice 1 wird automatisch ausgewählt, Kampf-Event feuert.

## Dringlichkeits-Sound hinzufügen

Für mehr Atmosphäre: PlaySound-Node vor der PlayerChoice:

```text
[SayLine: "Du hast 10 Sekunden..."]
   │
   ▼
[PlaySound: SE_TickingClock  Loop: true]
   │
   ▼
[PlayerChoice  Timeout: 10s ...]
```

Den Loop-Sound am Exit-Node stoppen (SideEffect oder über Event).

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Enemy_Ultimatum
   └─ ...
```

> 📸 **Bild-Platzhalter:** `timed-choice-ingame.png` — Preview-Runner mit sichtbarem Countdown-Timer.
> *Setup:* Preview-Runner offen. Choice-List zeigt zwei Choices. Über den Choices: Countdown-Balken / Zahl `8` (simuliert nach 2 Sekunden). Timer läuft sichtbar runter.

## Variation / Weiter gehen

- **Kurzer Timeout** (3s) für Panik-Entscheidungen: Spieler muss instinktiv reagieren.
- **Timeout ohne sichtbaren Timer**: `bShowCountdownInUI = false` → Spieler weiß nicht, dass die Zeit läuft. Horror-Muster.
- Timeout-Ergebnis in einem externen System verarbeiten: `OnDialogueEnded(ExitStatus == Failed)`.

## Troubleshooting

**Timer läuft nicht.**
`bEnableTimeout = false` oder `TimeoutDuration = 0`. Beides prüfen.

**Falscher Choice wird auto-gewählt.**
`DefaultChoiceIndex` ist 0-basiert. Index 0 = erste Choice, Index 1 = zweite Choice. Zähle die Choices im Details-Panel nach.

**Countdown erscheint nicht im Widget.**
Das Standard-Widget implementiert den Countdown automatisch. Bei eigenem UMG-Widget: `GetChoiceCountdown()` aus der Instance lesen und an den ProgressBar binden.
