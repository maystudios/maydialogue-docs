# Branch

Der **Branch-Node** ist die **automatische** Verzweigung: kein Spieler-Input, nur eine Bedingung.

## Runtime-Verhalten

`ExecuteNode`:

1. `Condition`-Requirement evaluieren.
2. Bei `Passed` → `Advance(TrueOutputGuid)`.
3. Bei `Failed*` → `Advance(FalseOutputGuid)`.
4. Wenn `bHasFallback` und das Ergebnis uneindeutig ist → `Advance(FallbackOutputGuid)`.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `Condition` | `UMayDialogueRequirement*` (Instanced) | Einzelne Bedingung. |
| `bHasFallback` | `bool` | Aktiviert den dritten Output-Pin. |

## Pins

* **Input**.
* **True** – wenn Condition passed.
* **False** – wenn Condition failed.
* **Fallback** (optional) – für degenerierte Fälle.

## Typisches Pattern

```
[SayLine: "..."]
  │
  ▼
[Branch: HasTag "Story.Met.Guard"]
  ├─ True  ──► [SayLine: "Schön, dich wiederzusehen."]
  └─ False ──► [SayLine: "Halt! Wer bist du?"]
```

## Unterschied zu PlayerChoice

| | Branch | PlayerChoice |
| --- | --- | --- |
| Treiber | Automatisch nach Condition | Spieler-Klick |
| Anzahl Bedingungen | 1 | 1 pro Choice |
| UI-Auftritt | Nein | Ja |
| Blockiert Instance | Nein (immediate) | Ja (WaitingForChoice) |

## Anmerkungen

* Eine Condition muss gesetzt sein – sonst Validator-Error.
* Der Fallback-Output ist als Sicherheitsnetz gedacht: *„falls die Condition weder klar Passed noch klar Failed ist"*. In der Praxis selten nötig, aber nützlich in GAS-Konstellationen, wo ein ASC fehlen könnte.
