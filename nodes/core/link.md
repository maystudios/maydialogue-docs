# Link

Der **Link-Node** springt in ein **anderes Dialog-Asset** – mit optionalem Return-Stack.

## Runtime-Verhalten

`ExecuteNode`:

1. Push Scope-Entry auf den Stack: `{ Asset: ThisAsset, ReturnNodeGuid: NextNodeAfterLink }`.
2. Wechsel zum Ziel-Asset; starte dessen Entry-Node.

Beim Erreichen eines Exit im Ziel-Asset:

* Scope-Stack pop.
* Wenn Stack nicht leer: zurück zum `ReturnNodeGuid`.
* Wenn Stack leer: Dialog endet komplett.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `TargetAsset` | `UMayDialogueAsset*` | Das Asset, in das gesprungen wird. |
| `bReturnAfterExit` | `bool` | Wenn `true`: Scope-Push (kehrt zurück). Wenn `false`: Ziel ersetzt komplett (kein Return). |

## Pins

* **Input**.
* **Output** – nächstes Node nach Return (nur wenn `bReturnAfterExit`).

## Typisches Pattern: Wiederverwendbare Fragmente

```
GreetingHub:        [SayLine: "Willkommen."] → [Link → DA_CommonFarewell] → [Exit]
                                                  ↓ ReturnPin
                                                  [SetVariable: HasGreeted = true]
                                                  ↓
                                                  [Exit]

DA_CommonFarewell:  [Entry] → [SayLine: "Gute Reise."] → [Exit]
```

Nach dem Farewell-Dialog springt die Ausführung via Scope-Stack zurück und führt die `SetVariable`-SideEffect aus.

## Typisches Pattern: Kapitel-Wechsel

```
[Exit_HubDialog] wird durch [Link → DA_Chapter2_Opening] ersetzt, bReturnAfterExit=false.
```

Das Hub-Dialog-Asset ist dadurch beim Übergang aus dem Scope.

## Anmerkungen

* **Scope-Stack-Tiefe**: beliebig tief schachtelbar.
* **Cross-Asset-Step-Into im PIE-Debugger ist rudimentär** – siehe [Debugger](../../editor/debugger.md#bekannte-einschrankungen).
* **Zirkulare Links** (A→B→A→B…) können theoretisch endlos laufen; im Zweifel durch ein Requirement / Variable gebremst.
