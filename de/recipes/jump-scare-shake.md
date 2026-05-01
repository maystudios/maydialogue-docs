---
description: Schreckmoment im Dialog mit Camera-Shake und NPC-Animation – atmosphärischer Horror ohne Cutscene.
---

# Jump-Scare mit Camera-Shake

## Szenario

Mitten im Gespräch macht der NPC eine aggressive Bewegung und die Kamera wackelt. Der Spieler kriegt einen Schreck, ohne dass ein Cutscene nötig ist. Das Rezept kombiniert **CameraShake**-Node, **PlayAnimation**-Node und einen kurzen Sound für den maximalen Effekt.

## Was du lernst

- CameraShake-Node mit einer `UCameraShakeBase`-Subklasse konfigurieren.
- PlayAnimation-Node für die NPC-Schreck-Montage einsetzen.
- PlaySound-Node für einen nicht-Voice-Stinger.
- Die Timing-Reihenfolge: erst Animation, dann Shake, dann Reaktions-SayLine.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- CameraShake-Asset vorhanden (Blueprint-Subklasse von `UCameraShakeBase`).
- Montage-Asset für NPC-Schreck-Bewegung.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: NPC – "Du solltest..." AdvanceMode: Manual]
   │
   ▼
[PlayAnimation: NPC – Montage: AM_NPC_LungeForward  WaitForEnd: false]
   │
   ▼
[CameraShake: BS_HorrorShake  Scale: 1.5  Radius: 500cm]
   │
   ▼
[PlaySound: SE_JumpScareStinger  2D: true]
   │
   ▼
[SayLine: NPC – "...NIEMALS zurückkommen!"]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `jump-scare-shake-graph-overview.png` — Horror-Dialog-Graph mit drei Action-Nodes für den Schreckmoment.
> *Setup:* Asset `DA_Horror_NPC_Threat` geöffnet. SayLine → PlayAnimation (orange Box) → CameraShake (rote Box) → PlaySound (gelbe Box) → SayLine → Exit. Alle drei Nodes prominent im Hauptfluss – leicht zu erkennen, was den Schreckmoment ausmacht.

## Schritt-für-Schritt

### 1. CameraShake-Asset erstellen

In UE: neues Blueprint mit Parent `UCameraShakeBase` (z.B. `WaveOscillatorCameraShake`). Beispiel-Settings für Horror:

| Property | Wert |
|----------|------|
| `OscillationDuration` | `0.5` |
| `RotOscillation.Pitch.Amplitude` | `3.0` |
| `RotOscillation.Yaw.Amplitude` | `2.0` |
| `LocOscillation.Z.Amplitude` | `5.0` |

Asset-Name: `BS_HorrorShake`.

### 2. PlayAnimation-Node konfigurieren

Vom SayLine-Output → **Create Node → Play Animation**:

| Property | Wert |
|----------|------|
| `TargetParticipantTag` | `Dialogue.Speaker.HorrorNPC` |
| `Montage` | `AM_NPC_LungeForward` |
| `StartSection` | `Default` |
| `bWaitForEnd` | `false` (Dialog-Flow sofort weiter, Animation läuft parallel) |

> 📸 **Bild-Platzhalter:** `jump-scare-shake-animation-details.png` — Details-Panel des PlayAnimation-Nodes.
> *Setup:* PlayAnimation-Node ausgewählt. Details: `TargetParticipantTag = Dialogue.Speaker.HorrorNPC`, `Montage = AM_NPC_LungeForward`, `bWaitForEnd = false`.

### 3. CameraShake-Node konfigurieren

Vom PlayAnimation-Output → **Create Node → Camera Shake**:

| Property | Wert |
|----------|------|
| `ShakeClass` | `BS_HorrorShake` |
| `Scale` | `1.5` |
| `PlaySpace` | `CameraLocal` |
| `bAttenuateWithRadius` | `true` |
| `Radius` | `500.0` |
| `TargetActor` | *(leer = immer spielen)* |

### 4. PlaySound-Node für den Stinger

Vom CameraShake-Output → **Create Node → Play Sound**:

| Property | Wert |
|----------|------|
| `Sound` | `SE_JumpScareStinger` |
| `b2DSound` | `true` |
| `VolumeMultiplier` | `1.2` |

`b2DSound = true` – der Stinger kommt nicht aus dem Raum, sondern direkt im Mix.

> 📸 **Bild-Platzhalter:** `jump-scare-shake-playsound-details.png` — PlaySound-Node mit 2D-Flag.
> *Setup:* PlaySound-Node ausgewählt. Details: `Sound = SE_JumpScareStinger`, `b2DSound = true (Checkbox)`, `VolumeMultiplier = 1.2`.

### 5. Reaktions-SayLine

Vom PlaySound-Output → SayLine *„...NIEMALS zurückkommen!"* → Exit.

### 6. Compile und PIE testen

Im PIE: Dialog starten. Bei der zweiten SayLine sollte die Montage laufen, Kamera wackeln, Stinger spielen – fast simultan durch die kurze Node-Ausführungszeit.

## Timing-Optimierung

Wenn Shake und Sound einen Frame zu früh kommen:
- SayLine vor dem Shake mit `AdvanceMode = Timer` und `AutoAdvanceDelay = 0.1` einfügen (Micro-Pause vor Shake).
- Oder PlayAnimation mit `bWaitForEnd = true` → Shake feuert erst wenn Montage fertig.

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Horror_NPC_Threat
   └─ ...
```

## Variation / Weiter gehen

- Kamera-Schwenk VOR dem Shake: [Kamera-Schwenk auf Sprecher](camera-pan-on-speaker.md) + dann Shake.
- NPC-Animation mit Spieler-Reaktion kombinieren → [NPC-Animation während Zeile](npc-animation-during-line.md).
- Shake über GameplayCue statt direkten Node: `TriggerCue`-Node → GC_HorrorShake.

## Troubleshooting

**CameraShake passiert nicht.**
Spieler-Kamera ist kein `APlayerCameraManager`-unterstütztes Kamera-System. Prüfe ob `GetPlayerCameraManager()` einen validen Wert gibt. Bei eigener Kamera-Logik: CameraShake manuell via `PlayCameraShake` im OnDialogueEvent-Handler triggern.

**Animation spielt, aber Shake kommt nicht.**
`bWaitForEnd = true` am PlayAnimation-Node und Montage sehr lang. Setze `bWaitForEnd = false`.

**Stinger zu laut.**
`VolumeMultiplier` reduzieren oder im Sound-Mix einen Ducking-Kanal für Dialog-Stinger konfigurieren.
