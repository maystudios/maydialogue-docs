# Camera Shake

Löst einen Kamera-Shake auf der Spieler-Kamera aus.

## Runtime-Verhalten

`ExecuteNode` ruft `PlayerController->ClientStartCameraShake(CameraShakeClass, Scale)` und gibt Advance zurück.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `CameraShakeClass` | `TSubclassOf<UCameraShakeBase>` | Beliebige UE-Camera-Shake-Klasse. |
| `Scale` | `float` | Shake-Intensität. Default: 1.0. |
| `bRadial` | `bool` | Räumlich begrenzt (Spieler muss nahe am Target sein). |
| `Radius` | `float` | Nur bei `bRadial`: Wirkungsradius. |

## Typisches Pattern

```
[SayLine: "Ein Schrei ertönt!"]
  │
  ▼
[CameraShake: Scale=2.0]
  │
  ▼
[SayLine: Spieler "Was war das?!"]
```

## Anmerkungen

* Horror-typische Nutzung: Jump-Scares, plötzliche Enthüllungen, Monster-Erscheinen.
* Der Shake überlagert sich mit normaler Kamera-Kontrolle – keine Konflikte mit CameraFocus.
