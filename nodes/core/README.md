# Core-Nodes

Die **neun** Core-Nodes sind das Rückgrat jedes Dialogs. Sie definieren den Fluss: Wo fängt es an, wer spricht, wo verzweigt es, wo endet es.

| Node | Kategorie | Advance-Modi |
| --- | --- | --- |
| [Entry](entry.md) | Struktur | Immediate (auto) |
| [Exit](exit.md) | Struktur | – (Ende) |
| [Say Line](say-line.md) | Präsentation | Manual / Timer / AfterVoice / AfterAnimation / Immediate |
| [Player Choice](player-choice.md) | Interaktion | – (wartet auf Choice) |
| [Branch](branch.md) | Flow | Immediate |
| [Random Line](random-line.md) | Flow + Präsentation | wie Say Line |
| [Wait](wait.md) | Flow | auf Duration / Event |
| [Link](link.md) | Flow | Immediate (oder wie Child-Dialog) |
| [SubGraph](sub-graph.md) | Flow | Immediate |

Jede Seite dokumentiert Properties, Input/Output-Pins, Beispiel und bekannte Einschränkungen.
