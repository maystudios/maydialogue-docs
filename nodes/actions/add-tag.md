# Add Tag

Fügt einen **LooseGameplayTag** zu einem ASC hinzu. Kein GameplayEffect-Overhead.

## Runtime-Verhalten

`ExecuteNode` (im MayDialogueGAS-Modul):

1. Löst Target-ASC auf (je nach `bAddToInstigator`).
2. `ASC->AddLooseGameplayTag(TagToAdd)`.
3. Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `TagToAdd` | `FGameplayTag` | Der Tag. |
| `bAddToInstigator` | `bool` | `true` → Spieler; `false` → Target. |

## Typisches Pattern

Dialog-State-Flag am NPC setzen:

```
[PlayerChoice: "Ich erzähle dir ein Geheimnis..."]
  │
  ▼
[AddTag: Tag=Story.Guard.KnowsSecret, bAddToInstigator=false]
  │
  ▼
[SayLine: NPC "Das darf niemand erfahren..."]
```

## LooseTag vs. GE-Applied Tag

| | LooseTag | GameplayEffect-Tag |
| --- | --- | --- |
| Persistent | Bis explizit entfernt | Bis GE endet / gestackt wird |
| Overhead | Sehr gering | GE-Spec + Modifier |
| Replikation | Via `Minimal`-Replication-Modus nicht perfekt | Full-Replication-Support |

Für Dialog-Flags *„hat mich gegrüßt"*, *„kennt mein Geheimnis"* ist LooseTag die richtige Wahl.

## Anmerkungen

* Partner: [Remove Tag](remove-tag.md).
* Kein Stack-Support: mehrmaliges Add ist idempotent (Tag ist drin oder nicht).
