---
description: Kamera automatisch auf den aktiven Sprecher schwenken – CameraFocus-Node Schritt für Schritt.
---

# Kamera-Schwenk auf Sprecher

## Szenario

Während eines Verhörs soll die Kamera bei jeder neuen Zeile auf den aktuellen Sprecher schwenken: erst auf den Detektiv, dann auf den Verdächtigen, dann zurück. Der **CameraFocus**-Node übernimmt das – er findet den richtigen Actor über den Participant-Tag und blendet die Kamera mit konfigurierbarer BlendTime hin.

## Was du lernst

- CameraFocus-Node anlegen und mit einem Participant-Tag verknüpfen.
- BlendTime und CameraOffset einstellen.
- Kamera am Dialog-Ende automatisch zurücksetzen.
- Mehrere Sprecher-Wechsel in einem Dialog choreografieren.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- NPC-Actor hat eine Participant-Komponente mit gesetztem `CameraTargetOffset`.

## Mini-Graph

```text
[Entry]
   │
   ▼
[CameraFocus: Speaker=Detective  BlendTime=0.5s]
   │
   ▼
[SayLine: Detektiv – "Wo waren Sie am Abend des 12ten?"]
   │
   ▼
[CameraFocus: Speaker=Suspect  BlendTime=0.5s]
   │
   ▼
[SayLine: Verdächtiger – "Zu Hause. Allein."]
   │
   ▼
[CameraFocus: Speaker=Detective  BlendTime=0.3s]
   │
   ▼
[SayLine: Detektiv – "Natürlich."]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `camera-pan-on-speaker-graph-overview.png` — Verhör-Dialog mit alternierenden CameraFocus-Nodes.
> *Setup:* Asset `DA_Interrogation_Alibi` geöffnet. Von links nach rechts: Entry → CameraFocus (grau-blaue Box, "Detective") → SayLine → CameraFocus ("Suspect") → SayLine → CameraFocus ("Detective") → SayLine → Exit. Nodes wechseln sichtbar zwischen zwei Speaker-Tags.

## Schritt-für-Schritt

### 1. Participant-CameraTargetOffset konfigurieren

Auf dem Detektiv-Actor: **Participant-Komponente → CameraTargetOffset** = `{X:0, Y:0, Z:170}` (Augenhöhe). Analog für den Verdächtigen.

### 2. CameraFocus-Node einfügen

Vom Entry-Output → **Create Node → Camera Focus**.

| Property | Wert |
|----------|------|
| `SpeakerTag` | `Dialogue.Speaker.Detective` |
| `BlendTime` | `0.5` |
| `CameraOffset` | `{X:-80, Y:30, Z:0}` (leichter Over-Shoulder) |
| `bRestoreOnDialogEnd` | `true` |

> 📸 **Bild-Platzhalter:** `camera-pan-on-speaker-focus-details.png` — Details-Panel des ersten CameraFocus-Nodes.
> *Setup:* Erster CameraFocus-Node ausgewählt. Details: `SpeakerTag = Dialogue.Speaker.Detective`, `BlendTime = 0.5`, `CameraOffset = (X=-80, Y=30, Z=0)`, `bRestoreOnDialogEnd = true`.

### 3. Dialogue-Graph aufbauen

Schema: **CameraFocus → SayLine** wiederholen für jeden Sprecher-Wechsel. Den zweiten CameraFocus auf `Dialogue.Speaker.Suspect` zeigen lassen.

### 4. Dritten CameraFocus zurück zum Detektiv

Kopiere den ersten CameraFocus-Node (Ctrl+C, Ctrl+V) und setze BlendTime auf `0.3` für einen schnelleren Rückschnitt.

### 5. Kamera-Reset am Dialog-Ende

`bRestoreOnDialogEnd = true` am ersten CameraFocus-Node → beim Exit-Node wird die Original-Kameraposition automatisch wiederhergestellt.

### 6. Compile und PIE testen

Im PIE: Dialog starten, Kamera schwenkt bei jedem CameraFocus-Node. Nach Exit: Kamera in Ausgangsposition.

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Interrogation_Alibi
   ├─ Instigator: Get Player Pawn
   └─ Target: Suspect Actor Reference
```

> 📸 **Bild-Platzhalter:** `camera-pan-on-speaker-bp-trigger.png` — Blueprint-Trigger mit beiden Actors als Teilnehmer.
> *Setup:* Spieler-Controller BP oder Interaktion-Komponente. `Start Dialogue`: `Instigator = Get Player Pawn`, `Target = Suspect Actor`. Beide Actors haben Participant-Komponenten.

{% hint style="info" %}
**C++-Variante**

```cpp
// Drei Actors: Player, Detektiv, Verdächtiger
TArray<AActor*> Participants = { PlayerPawn, DetectiveActor, SuspectActor };
Sub->StartDialogueMulti(Asset, Participants);
```
{% endhint %}

## CameraFocus als SideEffect

Wenn der Kamera-Schwenk ein Nebenschritt ist (nicht der Hauptschritt des Flows), hänge ihn als **SideEffect** an die SayLine:
- SayLine-Node → SideEffect → CameraFocus.
- Der Schwenk passiert beim Betreten der SayLine, nicht als eigener Node im Fluss.

## Variation / Weiter gehen

- **FOV-Override**: `FOVOverride = 70` am CameraFocus für dramatische Weitwinkel-Momente.
- **LevelSequence-Integration**: Für cineastische Einstellungen CameraFocus mit einer LevelSequence verknüpfen.
- **Jump-Scare**: CameraFocus + CameraShake kombinieren → [Jump-Scare mit Camera-Shake](jump-scare-shake.md).

## Troubleshooting

**Kamera schwenkt nicht.**
Participant-Komponente auf dem NPC-Actor fehlt oder `ParticipantTag` stimmt nicht mit `SpeakerTag` im CameraFocus-Node überein.

**Kamera springt statt zu blenden.**
`BlendTime = 0` oder das Kamera-System des Projekts überschreibt den Blend. Prüfe ob ein anderer Camera-Manager aktiv ist.

**Kamera kehrt nach Dialog nicht zurück.**
`bRestoreOnDialogEnd = false`. Setze das Flag auf `true` am CameraFocus-Node oder manuell nach `OnDialogueEnded`.
