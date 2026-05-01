# SubGraph

Der SubGraph-Node klappt einen eingebetteten Teil-Graphen innerhalb desselben Assets auf. Er funktioniert wie [Link](link.md), aber das Ziel lebt im gleichen Asset — kein separates Asset nötig.

## Wann setze ich ihn ein?

- Wenn ein Dialog-Fragment nur in diesem Asset vorkommt und nicht von anderen Assets geteilt wird.
- Um komplexe Graphen übersichtlich zu halten: wiederholte Sequenzen (Abschied, Kampfreaktion) einmal aufbauen und mehrfach aufrufen.
- Für verschachtelte Sub-Dialoge (Sub-Graph in Sub-Graph), ohne separate Assets anlegen zu müssen.
- Wenn du im Editor Breadcrumb-Navigation und Step-Into-Debugging bevorzugst (funktioniert für Sub-Graphs besser als für Cross-Asset-Links).

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `SubGraphName` | `FText` | leer | Anzeigename des Sub-Graphs im Nodes-Körper, z.B. `"Combat Reaction"`. |
| `bReturnAfterSubGraph` | `bool` | `true` | `true` = nach Exit des Sub-Graphs zurück zum nächsten Node; `false` = Sub-Graph ist terminal, kein Return. |

{% hint style="info" %}
**Öffnen:** Doppelklick auf den SubGraph-Node öffnet den eingebetteten Graphen als eigenen Tab mit Breadcrumb-Pfad. Zurück mit dem Breadcrumb oder der Tab-Liste.
{% endhint %}

> 📸 **Bild-Platzhalter:** `subgraph-node-graph.png` — SubGraph-Node im Haupt-Graphen.
> *Setup:* Haupt-Graph mit zwei SubGraph-Nodes: `"Painful Farewell"` (Input von `SayLine "Du hast mich betrogen!"`) und `"Happy Farewell"` (Input von `SayLine "Du bist ein Freund."`). Beide SubGraph-Nodes zeigen ihren Namen im Body. Output-Pins verbunden mit `Exit`.

> 📸 **Bild-Platzhalter:** `subgraph-inner-graph.png` — Geöffneter Sub-Graph "Painful Farewell" im eigenen Tab.
> *Setup:* Tab `DA_Abschied > Painful Farewell` geöffnet. Sichtbar: `Entry` → `SayLine "Das war ein Fehler."` → `CameraShake` → `Exit`. Breadcrumb oben zeigt `DA_Abschied / Painful Farewell`.

> 📸 **Bild-Platzhalter:** `subgraph-details-panel.png` — Details-Panel des SubGraph-Nodes.
> *Setup:* SubGraph-Node "Painful Farewell" auswählen. Im Details-Panel sichtbar: `SubGraphName = "Painful Farewell"`, `bReturnAfterSubGraph = true`.

## Mini-Beispiel

```text
Haupt-Graph:
  [SayLine: NPC | "Du hast mich betrogen!"]  ──► [SubGraph "Painful Farewell"] ──► [Exit: Failed]
  [SayLine: NPC | "Du bist ein Freund."]     ──► [SubGraph "Happy Farewell"]   ──► [Exit: Completed]

SubGraph "Painful Farewell":
  [Entry] → [SayLine: NPC | "Ich vergesse das nie."] → [CameraShake] → [Exit]

SubGraph "Happy Farewell":
  [Entry] → [SayLine: NPC | "Bis zum nächsten Mal."] → [PlayAnimation: HugMontage] → [Exit]
```

> 📸 **Bild-Platzhalter:** `subgraph-example-graph.png` — Haupt-Graph mit zwei SubGraph-Nodes plus geöffneter Sub-Graph im Hintergrund.
> *Setup:* Haupt-Tab zeigt zwei Pfade je mit einem SubGraph-Node. Zweiter Tab `Painful Farewell` geöffnet mit innerem Graph. Breadcrumb sichtbar.

## Häufige Fallstricke

- **`bReturnAfterSubGraph = false` und Output-Pin verbunden**: Das kann den Compiler verwirren. Wenn kein Return gewünscht ist, lasse den Output-Pin des SubGraph-Nodes unverbunden.
- **Sub-Graph ohne Exit-Node**: Der Sub-Graph hängt nach dem letzten Node. Immer einen Exit im Sub-Graph eintragen.

## SubGraph vs. Link

| | SubGraph | Link |
| --- | --- | --- |
| Ziel | Im selben Asset | Anderes Asset |
| Editor-Navigation | Tab mit Breadcrumb | Asset wechseln |
| Wiederverwendung | Nur in diesem Asset | Über mehrere Assets |
| Debugger Step-Into | Vollständig | Rudimentär |

**Faustregel:** Fragment nur in diesem Asset? SubGraph. Soll es mehrere Assets teilen? Link.
