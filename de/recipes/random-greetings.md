---
description: RandomLine-Node für frische NPC-Begrüßungen bei jedem Gespräch – mit Gewichtung und No-Repeat.
---

# Zufällige Begrüßungen

## Szenario

Eine Händlerin sagt bei jeder Interaktion eine andere Begrüßung aus einem Pool von fünf Sätzen. Eine davon soll seltener auftauchen und nur erscheinen, wenn die Händlerin den Spieler nicht gerade eben schon begrüßt hat. Ohne dieses Rezept würde jeder Dialog identisch klingen – mit ihm wirkt die Welt lebendig.

## Was du lernst

- RandomLine-Node anlegen und mit mehreren Einträgen befüllen.
- Weight-System nutzen, um seltene Zeilen zu gewichten.
- `bAllowRepeat = false` für automatisches No-Repeat aktivieren.
- Requirement an einem Entry für komplexere Filter nutzen.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.

## Mini-Graph

```text
[Entry]
   │
   ▼
[RandomLine: Merchant]
   ├─ "Hallo, Reisender."            (Weight 3)
   ├─ "Schön, dich zu sehen!"        (Weight 3)
   ├─ "Willkommen zurück."           (Weight 3)
   ├─ "Wie geht's?"                  (Weight 3)
   └─ "Dich habe ich ewig nicht..."  (Weight 1, Req: LastGreetingRecent=false)
   │
   ▼
[SayLine: "Wonach suchst du heute?"]
   │
   ▼
[PlayerChoice: Kaufoptionen ...]
```

> 📸 **Bild-Platzhalter:** `random-greetings-graph-overview.png` — Übersichtsgraph des Händler-Dialogs mit RandomLine.
> *Setup:* Asset `DA_Merchant_Greet` geöffnet. Von links nach rechts: `Entry` → `RandomLine`-Node (Würfel-Icon in Title, alle 5 Entries als Pills im Body sichtbar) → `SayLine "Wonach suchst du"` → `PlayerChoice`. Verbindungen klar, Entries mit Weight-Werten im Node-Body erkennbar.

## Schritt-für-Schritt

### 1. Asset und Speaker anlegen

Asset: `DA_Merchant_Greet`. Speaker: `DisplayName = Händlerin`, `SpeakerTag = Dialogue.Speaker.Merchant`.

### 2. RandomLine einfügen

Vom Entry-Output → **Create Node → Random Line**. Der Node zeigt ein Würfel-Icon und eine leere Eintrags-Liste.

### 3. Einträge befüllen

Im Details-Panel unter `LineEntries` viermal **Add Entry** klicken und ausfüllen:

| Text | Weight |
|------|--------|
| „Hallo, Reisender." | 3 |
| „Schön, dich zu sehen!" | 3 |
| „Willkommen zurück." | 3 |
| „Wie geht's?" | 3 |

Dann **Add Entry** für den fünften Eintrag: *„Dich habe ich ewig nicht gesehen."*, Weight = 1.

> 📸 **Bild-Platzhalter:** `random-greetings-randomline-details.png` — Details-Panel des RandomLine-Nodes mit allen Einträgen.
> *Setup:* RandomLine ausgewählt. Details-Panel zeigt: `LineEntries` mit 5 Einträgen, jeder mit Text und Weight. Letzter Eintrag Weight = 1, darunter Requirements ausgeklappt mit `CheckParticipantVariable: LastGreetingRecent = false`.

### 4. Seltene Zeile absichern

Am fünften Entry unter **Requirements → Add → CheckParticipantVariable**:

| Property | Wert |
|----------|------|
| `VariableName` | `LastGreetingRecent` |
| `ExpectedValue` | `false` |
| `FailureResult` | `FailedAndHidden` |

Wenn die Variable `true` ist (gerade gesprochen), wird der Eintrag vollständig ausgeblendet.

### 5. No-Repeat aktivieren

Im RandomLine-Node: `bAllowRepeat = false`. Das Plugin merkt sich den letzten gezogenen Index pro Participant und wählt beim nächsten Mal einen anderen.

### 6. Recent-Flag setzen (optional)

Am Entry-Node einen **SideEffect → SetVariable** hängen:
- `Variable`: `LastGreetingRecent`, Scope: `Participant`, Typ: `Bool`, Wert: `true`.

Am Exit-Node denselben SideEffect mit Wert `false` – oder über einen Timer im Participant-Blueprint nach 60 Sekunden zurücksetzen.

### 7. Fortsetzung verdrahten

RandomLine-Output → SayLine *„Wonach suchst du heute?"* → PlayerChoice (oder weiterer Gesprächsblock).

## Gewichtungs-Formel

```text
P(Entry_i) = Weight_i / Σ Weight_j    (nur sichtbare Entries)
```

Für unser Beispiel (alle sichtbar): seltene Zeile = `1 / (3+3+3+3+1) = 1/13 ≈ 7,7 %`.

Wenn die seltene Zeile durch das Requirement gefiltert ist, rechnet das System nur mit den verbleibenden vier Entries weiter – Gesamtgewicht passt sich automatisch an.

## Blueprint-Triggering

Kein besonderer Code nötig – normaler `Start Dialogue`-Call:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Merchant_Greet
   ├─ Instigator: Get Player Pawn
   └─ Target:     Self
```

> 📸 **Bild-Platzhalter:** `random-greetings-bp-trigger.png` — Blueprint-Trigger der Händlerin.
> *Setup:* Händlerin-Actor BP. `Event OnInteract` → `Start Dialogue`, Pins gefüllt.

## Variation / Weiter gehen

- RandomLine **mitten** im Dialog einbauen, nicht nur als Opener – funktioniert überall, wo eine SayLine passen würde.
- Weight dynamisch über eine Variable steuern: seltene Zeilen nach bestimmten Quest-Fortschritten häufiger machen.
- Den RandomLine-Block als Fragment auslagern und per Link aus mehreren NPC-Assets nutzen → [Wiederverwendbare Dialog-Fragmente](linking-dialogues.md).

## Troubleshooting

**Immer derselbe Entry wird gezogen.**
Im Preview-Runner wird ein deterministischer Seed verwendet – in PIE sieht das anders aus. Außerdem: wenn `bAllowRepeat = false` und nur ein Entry sichtbar ist (alle anderen durch Requirements gefiltert), gibt es keine Wahl.

**Seltene Zeile kommt trotz Weight 1 ständig.**
Alle anderen Entries sind durch Requirements ausgefiltert. Im Debugger prüfen, welche Entries sichtbar sind. Oft: Tippfehler im Variable-Namen des Requirement-Checks.

**RandomLine nimmt nie den Output.**
Prüfe, ob `bAdvanceAutomatically` am Node aktiviert ist. Wenn die inneren Entries `AdvanceMode = Manual` haben, wartet der Node auf einen Advance-Call, der nie kommt.
