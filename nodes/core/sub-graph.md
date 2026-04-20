# SubGraph

Der **SubGraph-Node** ist wie [Link](link.md), nur dass der Ziel-Graph **innerhalb desselben Assets** lebt.

## Runtime-Verhalten

`ExecuteNode`:

1. Push Scope-Entry.
2. Springt zum Entry-Node des Sub-Graphs (`SubGraphEntryGuid`, vom Compiler gesetzt).

Beim Sub-Graph-Exit: Pop, zurück zum Caller (wenn `bReturnAfterExit`).

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `SubGraphName` | `FText` | Anzeige im Graph-Node (z.B. *„Combat Reaction"*). |
| `SubGraphEntryGuid` | `FGuid` (compiled) | Zeigt auf den Entry-Node des Sub-Graphs. |
| `bReturnAfterExit` | `bool` | Wie beim Link. |

## Editor-Verhalten

* **Doppelklick** öffnet den Sub-Graph als Tab mit Breadcrumb-Pfad.
* Der Sub-Graph hat eigene Entry-/Exit-Nodes.
* Sub-Graphs können **beliebig geschachtelt** werden (Sub-Graphs in Sub-Graphs).

## Typisches Pattern: Wiederkehrende Reaktionen

```
Haupt-Dialog:
  [SayLine: "Du hast mich betrogen!"] → [SubGraph "Painful Farewell"] → [Exit]
  [SayLine: "Du bist ein Freund."]   → [SubGraph "Happy Farewell"]   → [Exit]

SubGraph "Painful Farewell":
  [Entry] → [SayLine: "..."] → [CameraShake] → [Exit]

SubGraph "Happy Farewell":
  [Entry] → [SayLine: "..."] → [PlayAnimation: HugMontage] → [Exit]
```

## Unterschied zu Link

| | Link | SubGraph |
| --- | --- | --- |
| Ziel | Anderes Asset | Eigener Sub-Graph innerhalb des Assets |
| Editor-Ansicht | Asset-Wechsel | Tab-Wechsel mit Breadcrumb |
| Asset-Kopplung | Lose (soft-ref) | Eng (im selben Asset) |
| Wiederverwendung | über mehrere Assets | nur innerhalb dieses Assets |

**Faustregel**: Wenn ein Fragment nur in **diesem** Asset wiederkehrt, nutze SubGraph. Wenn es auch **andere** Assets nutzen sollen: Link.

## Anmerkungen

* **Preview-Runner** navigiert automatisch in Sub-Graphs hinein und wieder heraus.
* **Debugger-Step-Into** funktioniert für Sub-Graphs sauber – im Gegensatz zu Cross-Asset-Link.
