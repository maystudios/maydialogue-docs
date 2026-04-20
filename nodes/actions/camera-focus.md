# Camera Focus

Blendet die Spieler-Kamera auf einen Ziel-Participant.

## Runtime-Verhalten

`ExecuteNode` startet einen Camera-Blend zum `CameraTarget` des Target-Participants und gibt sofort Advance zurück. Der Blend läuft parallel im Instance-Tick.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `TargetTag` | `FGameplayTag` | Welcher Participant wird fokussiert. |
| `BlendTime` | `float` | Blend-Dauer in Sekunden. Default: `DefaultCameraBlendTime` aus den Project-Settings. |
| `CameraOffset` | `FVector` | Zusätzlicher Offset relativ zum `CameraTargetOffset` des Participants. |
| `FOVOverride` | `float` | Wenn > 0: setzt FOV während des Focus. Restore bei Dialog-Ende. |
| `LevelSequence` | `ULevelSequence*` | Optional: zusätzliche LevelSequence, die während des Focus abgespielt wird. |

## Pins

Input / Output (Standard).

## Typisches Pattern

```
[SayLine: Wächter "Halt!"]
  │
  ▼
[CameraFocus: TargetTag=Dialogue.Speaker.Guard, BlendTime=0.5]
  │
  ▼
[SayLine: Wächter "Wer bist du?"]
```

## Anmerkungen

* Der **ursprüngliche FOV** wird global im Instance-Scope gehalten und bei Dialog-Ende restored – nicht pro Node. Wenn dein Dialog mehrere FOV-Overrides in Folge hat, stacken sie nicht (Backlog-Item 12).
* Ohne expliziten CameraFocus kann das Plugin **AutoFocusSpeaker** nutzen (Project-Setting), wenn aktiviert.
* Die optionale `LevelSequence` ist praktisch für cineastische Close-Ups – sie läuft parallel, blockiert Advance nicht.
