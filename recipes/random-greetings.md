# Zufällige Begrüßungen

NPCs, die immer dieselbe Zeile abspulen, wirken tot. Dieses Rezept zeigt dir, wie du mit der **RandomLine** pro Gespräch eine frische Begrüßung ziehst – inklusive Gewichtung und optionalem Memory, damit derselbe Satz nicht zweimal hintereinander kommt.

## Szenario

Eine Händlerin in der Stadt soll bei jeder Interaktion eine aus fünf möglichen Begrüßungen sagen. Eine davon („Dich habe ich ewig nicht gesehen.") soll seltener auftauchen und nur, wenn der Spieler *nicht* gerade eben mit ihr gesprochen hat.

## Beteiligte Nodes

* 1 × [Entry](../nodes/core/entry.md)
* 1 × [RandomLine](../nodes/core/random-line.md) mit 5 Entries
* 1 × [SayLine](../nodes/core/say-line.md) als Übergang zum eigentlichen Menü
* 1 × [PlayerChoice](../nodes/core/player-choice.md) oder weiter in einen SubGraph

## Graph-Mock-Up

```
[Entry]
   │
   ▼
[RandomLine]
   ├─ "Hallo, Reisender."            (Weight 3)
   ├─ "Schön, dich zu sehen!"        (Weight 3)
   ├─ "Willkommen zurück."           (Weight 3)
   ├─ "Wie geht's?"                   (Weight 3)
   └─ "Dich habe ich ewig nicht..."  (Weight 1, Req: NOT Recent)
   │
   ▼
[SayLine: "Wonach suchst du heute?"]
   │
   ▼
[PlayerChoice: ...]
```

## Schritt-für-Schritt

1. **Asset anlegen**: `DA_Merchant_Greet`. Speaker `Dialogue.Speaker.Merchant`.
2. **RandomLine einfügen**: Vom Entry-Output → *Create Node → Random Line*. Der RandomLine hat intern eine Liste `LineEntries` mit jeweils *Text*, *Voice*, *EmotionTags*, *Weight* und *Requirements*.
3. **Einträge füllen**: Klicke *Add Entry* viermal für die üblichen Zeilen (Weight = 3). Füge einen fünften Entry hinzu mit Weight = 1.
4. **Seltene Zeile absichern**: Am fünften Entry → *Requirements → Add → CheckParticipantVariable*:
   * `VariableName`: `LastGreetingRecent`
   * `ExpectedValue`: `false`
   * `FailureResult`: `FailedAndHidden`
5. **Fortsetzung verdrahten**: Der RandomLine hat einen einzigen Output – diesen zu einer kurzen SayLine *„Wonach suchst du heute?"* und dann zur PlayerChoice.
6. **SideEffect am Entry** einbauen (optional, für das *Recent*-Flag):
   * Am Entry-Node *SideEffects → Add → [SetVariable](../nodes/actions/set-variable.md)*
   * `Variable`: `LastGreetingRecent`, `Scope`: Participant, `Value`: `true`
7. **Reset-Mechanik**: An beliebiger Stelle (z.B. beim Exit) auf `false` setzen – oder über einen Timer im Participant-BP.
8. **Compile** und testen.

## RandomLine-Gewichtung

`Weight` ist ein relatives Gewicht. Die RandomLine zieht gewichtet:

```
P(Entry_i) = Weight_i / Σ Weight_j    (nur sichtbare Entries)
```

Für unser Beispiel (alle sichtbar) wäre die Wahrscheinlichkeit der seltenen Zeile `1 / (3+3+3+3+1) = 1/13 ≈ 7.7%`.

Wenn die seltene Zeile durch das Requirement ausgefiltert wird, rechnet das System nur mit den übrigen vier Entries weiter – das Gesamt-Weight passt sich automatisch an.

| Setting | Default | Anmerkung |
| --- | --- | --- |
| `Weight` | `1` | Muss > 0 sein. |
| `bAllowRepeat` | `false` | Verhindert, dass derselbe Entry wie beim letzten Mal gewählt wird (pro Participant). |
| `bAdvanceAutomatically` | `true` | Nach Abspielen automatisch zum Output gehen. |

## Runtime-Trigger

Kein besonderer Code nötig – ein ganz normaler `StartDialogue`-Call. Die Zufallsauswahl passiert innerhalb der Instance beim `ExecuteNode` der RandomLine.

```cpp
Sub->StartDialogue(DA_Merchant_Greet, Player, Merchant);
```

## Alternative: RandomLine als SayLine-Variante

Wenn du eine **einzelne** zufällige SayLine mitten im Gespräch einstreuen willst (nicht als Eröffnung), nimmst du die RandomLine genauso, aber nach einem anderen Node. Das funktioniert überall, wo ein SayLine passen würde.

```
[SayLine: "Also, du willst was kaufen?"]
   │
   ▼
[RandomLine: 3x Varianten von "Dann schau her..."]
   │
   ▼
[PlayerChoice: Kaufliste]
```

## Variante: Ohne Persistent Memory

Wenn dir das Recent-Flag zu kompliziert ist, lass das Requirement weg und akzeptiere, dass die seltene Zeile manchmal zweimal hintereinander kommt. Das ist für die meisten Projekte okay.

{% hint style="info" %}
**Pro-Tipp**: Für „Nie wieder dieselbe Zeile" brauchst du kein Requirement – setze `bAllowRepeat = false`. Das Plugin merkt sich den letzten Index automatisch im Participant-Memory.
{% endhint %}

## Troubleshooting

### Immer derselbe Entry wird gezogen

* Sitzt du im [Preview Runner](../editor/preview-runner.md)? Der nutzt einen deterministischen Seed, damit du reproduzierbar debuggen kannst. In PIE / Game sieht das anders aus.
* `bAllowRepeat` steht auf `false`, und es gibt nur einen sichtbaren Entry. Der Rest ist durch Requirements ausgefiltert.

### Die seltene Zeile kommt trotz `Weight = 1` ständig

* Alle anderen Entries sind durch Requirements ausgefiltert. Prüfe die Outline / den Debugger auf Filter-Rejects.
* Tippfehler im Variable-Namen des CheckParticipantVariable-Requirements – es greift ins Leere und liefert Passed.

### RandomLine nimmt nie den Output

* Das ist meistens ein Zusammenspiel-Bug mit `AdvanceMode = Manual` an den inneren Entries. Stelle sicher, dass die einzelnen Entry-Zeilen einen sinnvollen Advance-Mode haben.

## Nächster Schritt

* [Wiederverwendbare Dialog-Fragmente](linking-dialogues.md) – den RandomLine-Block als eigenes Asset und per Link aus jedem NPC-Gespräch referenzieren.
* [GAS-getriebener Dialog](gas-driven-dialogue.md) – eine der Begrüßungen stattdessen an einen konkreten GameplayTag koppeln (*„Ich sehe, du hast den Talisman!"*).
