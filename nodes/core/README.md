# Core-Nodes

Die neun Core-Nodes sind das Rückgrat jedes Dialogs. Sie legen fest, wo der Dialog startet, wer spricht, wo er verzweigt und wie er endet.

| Node | Wofür | Advance-Modus |
| --- | --- | --- |
| [Entry](entry.md) | Startpunkt des Graphen | Immediate (automatisch) |
| [Exit](exit.md) | Endpunkt mit Status | — (beendet Dialog) |
| [Say Line](say-line.md) | Sprecher sagt eine Zeile | Manual / Timer / AfterVoice / AfterAnimation / Immediate |
| [Player Choice](player-choice.md) | Spieler wählt eine Option | — (wartet auf Auswahl) |
| [Branch](branch.md) | Automatische Bedingungsverzweigung | Immediate |
| [Random Line](random-line.md) | Zufällige Zeile aus mehreren Varianten | wie Say Line |
| [Wait](wait.md) | Pause auf Zeit, Event oder Bedingung | auf Trigger |
| [Link](link.md) | Wechsel in anderes Asset | Immediate / wie Ziel-Dialog |
| [SubGraph](sub-graph.md) | Aufklappen eines internen Sub-Graphen | Immediate |

> 📸 **Bild-Platzhalter:** `core-overview-graph.png` — Übersicht aller Core-Nodes in einem Demo-Graphen.
> *Setup:* Neues Test-Asset mit folgenden Nodes von links nach rechts: `Entry` (grün) → `SayLine` → `PlayerChoice` mit zwei Choices → je eine `SayLine` → `Exit Completed` (oben) und `Exit Failed` (unten). Daneben separat: `Branch`-, `Wait`-, `RandomLine`-, `Link`- und `SubGraph`-Node nebeneinander, jeweils unverbunden, zur visuellen Übersicht.

Jede Seite dokumentiert Properties, Laufzeit-Verhalten, Beispiel-Graphen und typische Fallstricke.
