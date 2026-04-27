---
description: Innerer Monolog des Spielers mit 2D-Audio – kein räumlicher Sound, sondern direkt im Kopf.
---

# Innerer Monolog mit 2D-Audio

## Szenario

Der Spieler findet ein Tagebuch und liest innerlich dessen Inhalt. Die Stimme soll nicht aus dem Raum kommen, sondern wie ein Gedanke – trocken, nah, ohne 3D-Attenuation. Dafür nutzt du eine SayLine mit dem Spieler als Sprecher und einem Audio-Override auf 2D.

## Was du lernst

- SayLine mit dem Spieler als Sprecher konfigurieren.
- 2D-Audio-Override am Speaker einstellen (kein Attenuation-Asset).
- Kein NPC als Gesprächspartner: Dialog startet ohne Target.
- Typewriter-Effekt für Gedanken-Stil anpassen.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- Voice-Asset für Inneren Monolog importiert: `SW_InnerVoice_Line01`.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Spieler (Innere Stimme) – "Das war er. Der, von dem alle reden."  AdvanceMode: AfterVoice]
   │
   ▼
[SayLine: Spieler (Innere Stimme) – "Ich sollte vorsichtig sein."           AdvanceMode: Timer 3s]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `inner-monologue-2d-graph-overview.png` — Graph des Tagebuch-Monologs.
> *Setup:* Asset `DA_Diary_Page1` geöffnet. Entry → zwei SayLines (Title-Bar dunkelblau = Spieler-Sprecher) → Exit. Im Speakers-Panel sichtbar: Speaker "Innere Stimme" mit 2D-Override-Einstellung.

## Schritt-für-Schritt

### 1. Speaker "Innere Stimme" anlegen

Asset: `DA_Diary_Page1`. **Speakers-Panel** → **Add Speaker**:

| Feld | Wert |
|------|------|
| `DisplayName` | *(leer oder "Gedanken")* |
| `SpeakerTag` | `Dialogue.Speaker.PlayerInner` |
| `Color` | Dunkelblau oder Dunkelgrau |
| `AudioOverride.AttenuationAsset` | *(leer lassen = 2D)* |
| `AudioOverride.b2DAudio` | `true` |

Der leere `AttenuationAsset`-Slot zusammen mit `b2DAudio = true` sorgt dafür, dass der Voice-Clip als 2D-Sound gespielt wird – direkt im Mix, ohne räumliche Positionierung.

> 📸 **Bild-Platzhalter:** `inner-monologue-2d-speaker-settings.png` — Speakers-Panel mit 2D-Audio-Override.
> *Setup:* Speakers-Panel. Speaker "Innere Stimme": `SpeakerTag = Dialogue.Speaker.PlayerInner`, Farb-Chip dunkelblau, `AudioOverride` ausgeklappt: `b2DAudio = true`, `AttenuationAsset = None`.

### 2. SayLines konfigurieren

Erste SayLine: `SpeakerTag = Dialogue.Speaker.PlayerInner`, `DialogueText = "Das war er..."`, `AdvanceModeOverride = AfterVoice`, Voice-Asset: `SW_InnerVoice_Line01`.

Zweite SayLine: `AdvanceModeOverride = Timer`, `AutoAdvanceDelay = 3.0`.

### 3. Typewriter-Stil anpassen

Für Gedanken passt oft ein langsamerer Typewriter (nachdenklicher Effekt). Im SayLine-Node unter `TypewriterOverride`:
- `CharactersPerSecond = 30` (statt Default 60).
- `bSkipTypewriterOnAdvance = false` (damit der Text nicht sofort aufgedeckt wird).

### 4. Dialog ohne Target starten

Innere Monologe haben keinen NPC als Gesprächspartner. Der Start-Call gibt nur den Instigator mit:

```text
[Event OnTriggerEnter (Tagebuch-Volumen)]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Diary_Page1
   ├─ Instigator: Get Player Pawn
   └─ Target:     (leer / None)
```

> 📸 **Bild-Platzhalter:** `inner-monologue-2d-bp-trigger.png` — Blueprint mit Trigger-Volume und Dialogue-Start ohne Target.
> *Setup:* TriggerBox-Actor. `On Actor Begin Overlap` (mit Player-Tag-Prüfung) → `Start Dialogue`. `Target = None` sichtbar am Pin (unverbunden).

{% hint style="info" %}
**C++-Variante**

```cpp
// Tagebuch-Actor, wenn Spieler in den Overlap-Bereich tritt:
if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
{
    Sub->StartDialogue(DiaryAsset, PlayerPawn, nullptr); // kein Target
}
```
{% endhint %}

## DisplayName für Innere Stimme

Für Gedanken-Stil oft gewünscht: kein Name im Widget anzeigen. Zwei Möglichkeiten:
- `DisplayName` im Speaker leer lassen → Widget zeigt keinen Namen.
- Im Dialog-Widget: `bHideSpeakerName` für Spieler-Speaker setzen (Widget-Konfiguration).

## Variation / Weiter gehen

- Monolog mit **Kamera-Effekt** verbinden: CameraFocus-Node auf den Spieler oder einen Umgebungspunkt → [Kamera-Schwenk auf Sprecher](camera-pan-on-speaker.md).
- **Mehrere Seiten**: Separate Assets für jede Tagebuchseite + Link-Nodes für sequenzielles Lesen.
- Mix: Erste Zeile 2D (Gedanke), zweite Zeile 3D (laut sprechen) → zwei Speaker mit unterschiedlichem Audio-Override.

## Troubleshooting

**Voice klingt räumlich, nicht 2D.**
`b2DAudio` am Speaker nicht aktiviert oder `AttenuationAsset` versehentlich befüllt. Speakers-Panel prüfen.

**Kein Sound beim AfterVoice-Modus.**
Voice-Asset am SayLine-Node nicht zugewiesen. `AdvanceMode = AfterVoice` mit leerem Voice-Slot fällt auf Typewriter-Ende zurück – Zeile geht weiter, wenn Typewriter fertig ist.

**Dialog startet, aber Widget erscheint nicht.**
Widget braucht einen Instigator um zu wissen, wohin es rendert. Prüfe ob `Get Player Pawn` einen validen Wert zurückgibt.
