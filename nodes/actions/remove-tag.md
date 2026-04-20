# Remove Tag

Entfernt einen **LooseGameplayTag** von einem ASC.

## Runtime-Verhalten

`ExecuteNode` (im MayDialogueGAS-Modul):

1. Löst Target-ASC auf.
2. `ASC->RemoveLooseGameplayTag(TagToRemove)`.
3. Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `TagToRemove` | `FGameplayTag` | Der Tag. |
| `bRemoveFromInstigator` | `bool` | `true` → Spieler; `false` → Target. |

## Typisches Pattern

Reset eines früheren Flags:

```
[SayLine: "Du hast dich bewährt. Ich vertraue dir wieder."]
  │
  ▼
[RemoveTag: Tag=Story.Guard.Distrusts]
  │
  ▼
[Exit]
```

## Anmerkungen

* Entfernt **nur** LooseGameplayTags (per `AddLooseGameplayTag` gesetzt). GameplayEffect-granted Tags bleiben unberührt – die werden durch Effect-Ende / Stack-Modifikation entfernt.
* Idempotent: Kein Fehler, wenn der Tag gar nicht gesetzt war.
