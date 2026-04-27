---
description: Einzelne Zeilen oder Sound-Events feintunen – 2D-Override für innere Monologe, Volume und Pitch pro PlaySound.
---

# Node-Overrides

## Wann brauche ich das?

Node-Overrides sind für **Ausnahmen**. Du hast bereits Speaker-Overrides gesetzt, aber eine einzelne Zeile muss anders klingen:

- Diese eine SayLine ist innerer Monolog → 2D statt 3D
- Dieser Ambient-Sound soll leiser als normal sein
- Dieser Schrei soll höher klingen als der Rest

Alles, das **nur für diese eine Zeile gilt**, gehört auf den Node. Alles, das für den Charakter generell gilt, gehört auf den [Speaker-Override](speaker-overrides.md).

## Overrides auf SayLine

> 📸 **Bild-Platzhalter:** `node-overrides-sayline-details.png` — Details-Panel einer SayLine mit NodeAudioMode = Force2D.
> *Setup:* SayLine-Node im Graph-Editor auswählen. Details-Panel rechts zeigt: `NodeAudioMode = Force2D` (Dropdown, ausgewählt). Darüber `SpeakerTag`, `DialogueText`, darunter weitere SayLine-Properties. Roter Pfeil auf `NodeAudioMode`.

| Property | Typ | Verfügbar | Wirkung |
|---|---|---|---|
| `NodeAudioMode` | Enum | Ja | `Default` / `Spatial3D` / `Force2D` für diese Zeile |
| `VolumeMultiplier` | float | Derzeit nicht (geplant) | — |
| `PitchMultiplier` | float | Derzeit nicht (geplant) | — |

{% hint style="info" %}
Volume- und Pitch-Overrides pro SayLine sind noch in Arbeit. Workaround: Lege einen eigenen Sprecher-Eintrag an (z.B. `Dialogue.Speaker.InnerVoice`) mit den gewünschten Werten und nutze diesen Sprecher für die betreffenden Zeilen.
{% endhint %}

## Overrides auf PlaySound

Der `PlaySound`-Node kann eigenständig Audio abspielen – unabhängig von SayLines. Er hat vollständige Override-Kontrolle.

> 📸 **Bild-Platzhalter:** `node-overrides-playsound-details.png` — Details-Panel eines PlaySound-Nodes mit ausgefüllten Werten.
> *Setup:* PlaySound-Node im Graph auswählen. Details-Panel rechts zeigt: `Sound = SFX_Heartbeat`, `VolumeMultiplier = 0.4`, `PitchMultiplier = 1.0`, `bForce2D = false`, `TargetTag = Dialogue.Speaker.Guard`. Alle Felder ausgefüllt, roter Pfeil auf `VolumeMultiplier`.

| Property | Typ | Wirkung |
|---|---|---|
| `Sound` | USoundBase | Der abzuspielende Sound (SoundWave, SoundCue, MetaSound) |
| `VolumeMultiplier` | float | Volume-Skala für diesen Sound |
| `PitchMultiplier` | float | Pitch-Skala für diesen Sound |
| `bForce2D` | bool | 2D-Wiedergabe (ignoriert Attenuation) |
| `TargetTag` | FGameplayTag | Optional: Sound am Actor dieses Sprechers abspielen |

## Advance-Mode und Audio-Timing

Der `AdvanceModeOverride` auf einer SayLine beeinflusst, wie der Dialog-Flow mit dem Audio-Ende umgeht:

| Modus | Audio-Verhalten |
|---|---|
| `Manual` | Audio läuft, Spieler kann unabhängig weiterklicken |
| `Timer` | Dialog advanced nach `AutoAdvanceDelay`, unabhängig vom Audio |
| `AfterVoice` | Dialog advanced **genau** wenn das Voice-Asset zu Ende ist |
| `AfterAnimation` | Dialog advanced wenn die Montage endet (Audio läuft parallel) |
| `Immediate` | Kein Warten – Audio startet, wird aber ggf. sofort vom nächsten Node abgelöst |

`AfterVoice` ist der richtige Modus wenn Audio und Dialog-Flow synchron bleiben sollen.

## Praxisbeispiele

### Innerer Monolog – 2D statt 3D

```text
SayLine "Ich sollte fliehen."
  NodeAudioMode = Force2D
```

Diese Zeile spielt direkt ins Ohr des Spielers, ohne räumliche Positionierung – korrekt für Gedanken, die nur der Spieler hört.

> 📸 **Bild-Platzhalter:** `node-overrides-inner-monologue-graph.png` — Graph-Ausschnitt mit SayLine "Ich sollte fliehen." und NodeAudioMode = Force2D im Details-Panel.
> *Setup:* Graph-Editor mit SayLine-Node ausgewählt. Node-Title zeigt Sprecher und Text. Details-Panel rechts: `NodeAudioMode = Force2D` aufgeklappt. Im Hintergrund weitere Nodes des Dialogs erkennbar. Pfeil sichtbar: vorheriger Node → `SayLine "Ich sollte fliehen."` → nächster Node.

### Ambient-Sound leiser

```text
PlaySound
  Sound           = SFX_AmbientDrip
  VolumeMultiplier = 0.4
  bForce2D        = false
```

Der Tropfen-Sound soll im Hintergrund bleiben, nicht dominieren.

### Panischer Schrei – höhere Tonlage

```text
PlaySound
  Sound            = SFX_Scream
  PitchMultiplier  = 1.3
  VolumeMultiplier = 1.1
```

Höhere Frequenz unterstreicht die Dringlichkeit des Moments.

## Node-Override vs. Speaker-Override

```text
Faustregel: Gilt es nur für diese eine Zeile → Node-Override.
            Gilt es für den Charakter generell → Speaker-Override.
```

| Situation | Richtige Ebene |
|---|---|
| Alle Zeilen des Geistes klingen verhallt | Speaker-Override |
| Nur der Abschlusssatz des Geistes ist 2D | Node-Override (NodeAudioMode) |
| Hintergrund-Sound soll leiser sein | PlaySound mit VolumeMultiplier |
