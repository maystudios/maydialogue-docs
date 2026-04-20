# Trigger Cue

Feuert einen **GameplayCue** one-shot. Für cosmetische Akzente (Partikel, SFX, UI-Flashes).

## Runtime-Verhalten

`ExecuteNode` (im MayDialogueGAS-Modul):

1. Baut `FGameplayCueParameters` (Instigator, EffectCauser).
2. Löst Target-ASC auf.
3. `ASC->ExecuteGameplayCue(CueTag, Params)`.
4. Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `CueTag` | `FGameplayTag` | Muss unter `GameplayCue.*` liegen. |
| `bTriggerOnInstigator` | `bool` | `true` → Spieler; `false` → Target. |

## Typisches Pattern

Fluch-Visualeffekt, wenn der NPC ihn ausspricht:

```
[SayLine: NPC "Ich verfluche dich!"]
  │
  ▼
[TriggerCue: CueTag=GameplayCue.Dialog.Curse, bTriggerOnInstigator=false]
  │
  ▼
[SayLine: NPC "Mögen die Götter dir nicht mehr folgen."]
```

## Unterschied zu PlaySound / CameraShake

| | TriggerCue | PlaySound | CameraShake |
| --- | --- | --- | --- |
| System | GAS | UE-Audio | UE-Camera |
| Wird repliziert | Ja (via GAS) | Nein | Ja (Client-Start) |
| Typischer Use-Case | Kombinierter Partikel + Sound + UI-Flash | Nur Sound | Nur Kamera |

## Anmerkungen

* Cues mit `GameplayCueNotify_Static` (one-shot) passen am besten. Persistente Cues (`Notify_Actor`) brauchen explizites Adden/Entfernen über GE.
* Der Cue läuft **am Target-Actor** (bei `bTriggerOnInstigator=false`), nicht am Spieler-Camera-Space.
