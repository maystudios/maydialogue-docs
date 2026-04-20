# Play Sound

Spielt einen **Non-Voice-Sound** ab. Für Ambient-Akzente während eines Dialogs (Donner, Türknarren, Herzschlag).

## Runtime-Verhalten

`ExecuteNode`:

1. Löst den Target-Participant (wenn vorhanden) auf.
2. 3D-Playback am Participant oder 2D (je nach `bForce2D`).
3. Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `Sound` | `USoundBase*` | SoundCue / SoundWave / MetaSound. |
| `TargetTag` | `FGameplayTag` | Optional: Participant, an dessen Position gespielt wird. |
| `VolumeMultiplier` | `float` | Default: 1.0. |
| `PitchMultiplier` | `float` | Default: 1.0. |
| `bForce2D` | `bool` | Ignoriere 3D-Setup, spiele 2D. |

## Typisches Pattern

Donnerschlag zwischen zwei Zeilen:

```
[SayLine: "Ich habe ein schlechtes Gefühl..."]
  │
  ▼
[PlaySound: S_Thunder, bForce2D=true]
  │
  ▼
[SayLine: "...und es wird nur schlimmer."]
```

## Anmerkungen

* Voice-Lines gehören auf den SayLine-Node, nicht auf PlaySound. PlaySound ist für **alles außer Sprache**.
* Die 3-Level-Audio-Hierarchie (Plugin-Default → Speaker-Override → Node) greift für PlaySound-Nodes ebenfalls.
* Lange Sounds (mehrere Sekunden) sollten als Fire-and-Forget laufen – der Dialog advanced sofort, der Sound spielt weiter.
