# Play Animation

Spielt eine `UAnimMontage` auf einem Participant.

## Runtime-Verhalten

`ExecuteNode`:

1. Zielt auf den Participant mit `AnimationTargetTag`.
2. Holt dessen AnimInstance.
3. Spielt `Montage` mit `PlayRate`, optional ab `StartSection`.
4. Wenn `bWaitForMontageEnd`: Advance ist async – der Node wartet auf `OnMontageEnded`.
5. Sonst: Fire-and-Forget, sofortiges Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `AnimationTargetTag` | `FGameplayTag` | Welcher Participant spielt. |
| `Montage` | `UAnimMontage*` | Montage-Asset. |
| `PlayRate` | `float` | Geschwindigkeit. Default: 1.0. |
| `StartSection` | `FName` | Start-Section innerhalb der Montage (leer = ab Anfang). |
| `bWaitForMontageEnd` | `bool` | Dialog wartet auf Montage-Ende. |

## Typisches Pattern

Kopfnicken beim Zustimmen:

```
[PlayerChoice: "Ich helfe dir."]
  │
  ▼
[PlayAnimation: NPC nickt, bWaitForMontageEnd=false]
  │
  ▼
[SayLine: NPC "Danke."]
```

## Anmerkungen

* Bei **Abort** des Dialogs während einer wartenden Montage muss der Cleanup den `OnMontageEnded`-Delegate unbinden – aktuell ein bekannter Bug (Backlog-Item 3). Bis zum Fix: halte Animationen kurz oder nutze `bWaitForMontageEnd=false` in risikoreichen Pfaden.
* Validator warnt, wenn `bWaitForMontageEnd=true` und kein Output-Pin verbunden ist.
