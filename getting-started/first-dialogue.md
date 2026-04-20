# Ein vollständiger Walkthrough

Der [Quick Start](quick-start.md) hat einen minimalen Dialog gebaut. In diesem Walkthrough gehst du die nächste Stufe: **Variablen, Requirements, GAS-Tags, Branching, Random-Lines, Sub-Graphs, Persistence**. Am Ende hast du ein realistisches NPC-Gespräch, wie es in einem Horror- oder RPG-Spiel vorkommen könnte.

## Szenario

Du triffst einen Wächter vor einer Festung. Drei Faktoren bestimmen den Dialog:

1. **Hast du ihn schon einmal getroffen?** (Participant-Variable `HasMet: bool`).
2. **Hast du das Passwort gehört?** (GameplayTag `Story.Secret.HeardPassword` auf dem Spieler).
3. **Wie steht deine Reputation?** (GAS-Attribut `Reputation.Guards`).

## Schritt 1 · Neues Dialog-Asset

Lege `DA_Gate_Guardian` an und öffne es im Editor.

### Sprecher

| Tag | DisplayName | Farbe |
| --- | --- | --- |
| `Dialogue.Speaker.Guard` | Wächter | Dunkelrot |
| `Dialogue.Speaker.Player` | Du | Grau |

### Participant-Variablen

Im **Variables-Panel** eine Participant-Scope-Variable anlegen:

| Name | Typ | Scope | Default |
| --- | --- | --- | --- |
| `HasMet` | Bool | Participant | `false` |

## Schritt 2 · Begrüßung mit Branch

Nach dem Entry kommt ein **Branch**, der die Variable abfragt.

1. **Branch-Node** platzieren, Entry-Output verbinden.
2. Branch-Sub-Node hinzufügen: **MayDialogueRequirement_CheckParticipantVariable** (kommt mit dem Plugin – oder nutze das Projekt-eigene `HasMet`-Requirement).

{% hint style="info" %}
Wenn du im Quick-Start-Zustand noch keine eigene Variable-Check-Requirement hast, geht das Pattern genauso mit einer GAS-Tag-Abfrage: Requirement `HasTag` auf `Story.Met.Guard`.
{% endhint %}

3. True-Ausgang → **SayLine**: SpeakerTag Guard, Text `"Du bist also wieder hier. Was willst du?"`.
4. False-Ausgang → **SayLine**: SpeakerTag Guard, Text `"Halt! Wer bist du?"`.

An den Nodes nach der Begrüßung hängst du jeweils einen **SideEffect**-Sub-Node `SetVariable`, der `HasMet` auf `true` setzt.

## Schritt 3 · Player-Choice mit Requirement

Jede Variante mündet in einem PlayerChoice mit drei Optionen:

1. `"Ich kenne das Passwort: Elenderion."` – **nur wenn** der Spieler `Story.Secret.HeardPassword` hat.
2. `"Ich bin ein Freund des Königs."`
3. `"Das geht dich nichts an."`

### Requirement auf Choice 1

1. Choice 1 auswählen.
2. Sub-Node **HasTag-Requirement** hinzufügen:
   * `RequiredTag`: `Story.Secret.HeardPassword`.
   * `bCheckOnInstigator`: `true` (wir prüfen den Spieler, der den Dialog gestartet hat).
   * `bHideOnFail`: `true` – die Option soll komplett verschwinden, nicht bloß ausgegraut sein.

Das ergibt das RPG-Muster: *„Erst wenn du das Passwort gehört hast, siehst du die Option überhaupt."*

### Reputation-Abhängigkeit auf Choice 2

Choice 2 (`"Freund des Königs"`) bekommt ein **CheckAttribute**-Requirement:

* `Attribute`: `Reputation.Guards`.
* `ComparisonOp`: `>=`.
* `ComparisonValue`: `50`.
* `bCheckOnInstigator`: `true`.
* `FailureResult`: `FailedButVisible` (man sieht die Option, kann sie aber nicht wählen, Tooltip erklärt warum).
* `UnavailableReason`: `"Deine Reputation bei den Wachen ist zu niedrig."`

## Schritt 4 · Konsequenzen als Action-Nodes

Jede Choice hat eine prominente Action, die im Graph sichtbar sein soll.

### Choice 1 (Passwort)

Pfad: `PlayerChoice → ApplyEffect-Node → SayLine "Passiere in Frieden" → Exit(Completed)`.

* **ApplyEffect-Node**:
  * `EffectClass`: `GE_GuardTrust` (Projekt-eigener GameplayEffect, der die Reputation erhöht).
  * `EffectLevel`: `1.0`.
  * `bApplyToInstigator`: `true`.

### Choice 2 (König)

Pfad: `PlayerChoice → SayLine "Dann passiere" → Exit(Completed)`.

### Choice 3 (Frech)

Pfad: `PlayerChoice → AddTag-Node → CameraShake-Node → SayLine "Verzieh dich!" → Exit(Failed)`.

* **AddTag-Node**: Tag `Story.Guard.Hostile`, `bAddToInstigator = false` (auf den Wächter).
* **CameraShake-Node**: Projekt-eigener `UCameraShakeBase`, Scale `1.5`.

## Schritt 5 · Random-Greeting für wiederholte Besuche

Die SayLine `"Du bist also wieder hier."` wird nach dem dritten Besuch langweilig. Ersetze sie durch eine **RandomLine**:

1. Node durch **Random Line** ersetzen.
2. Lines:
   * `"Du bist also wieder hier. Was willst du?"`
   * `"Zurück? Hoffe du hast einen guten Grund."`
   * `"Noch einmal. Und heute?"`
3. `bRememberSelection`: `true` → keine Wiederholung hintereinander.

## Schritt 6 · Sub-Graph für Reputation-Feedback

Beide Choice-1- und Choice-2-Pfade enden mit einer fast identischen Verabschiedung. Lagere sie in einen Sub-Graph aus:

1. Rechtsklick → **SubGraph** anlegen, Name `SG_GuardFarewell`.
2. Doppelklick öffnet den Sub-Graph mit eigenem Breadcrumb.
3. Innen: `Entry → SayLine "Pass auf dich auf." → Exit`.
4. Vom Haupt-Graph rufen ApplyEffect- und SayLine-Nodes diesen Sub-Graph per **Link**- oder **SubGraph**-Node auf.

## Schritt 7 · Compile & Test

Compile. Prüfe die **Compiler Results**. Dann:

1. PIE starten.
2. Ohne Passwort & mit niedriger Reputation: Choice 1 fehlt, Choice 2 ist gegraut, Choice 3 funktioniert.
3. Mit Passwort (im Spieler-ASC Tag `Story.Secret.HeardPassword`): Choice 1 erscheint, ApplyEffect erhöht Reputation.
4. Nach erstem Gespräch ist `HasMet = true`; Dialog beginnt mit der Wiederkehrer-Variante.

## Schritt 8 · Persistence

Damit `HasMet` auch nach Level-Wechsel erhalten bleibt:

1. Beim Level-Verlassen: `UMayDialogueSaveHelper::QuickSaveToSlot("AutoSave", 0)` aufrufen.
2. Beim Level-Betreten: `UMayDialogueSaveHelper::QuickLoadFromSlot("AutoSave", 0)` aufrufen.

Details siehe [Persistence → QuickSave-Helper](../persistence/quicksave-helper.md).

## Zusammenfassung

Was du gebaut hast, nutzt fast jedes Plugin-Feature:

| Feature | Eingesetzt in |
| --- | --- |
| Branch | Begrüßungs-Verzweigung |
| Player Choice mit Requirements | GAS-Tag- & Attribut-gesteuerte Optionen |
| Action-Nodes | ApplyEffect, AddTag, CameraShake |
| Sub-Nodes | HasTag, CheckAttribute, SetVariable (SideEffect) |
| Variables | HasMet (Participant-Scope, persistent) |
| Random Line | Wiederholter Greeting |
| SubGraph | Geteilte Verabschiedung |
| Persistence | QuickSaveHelper |
| GAS | GE_GuardTrust, Tag-Abfragen, Attribut-Abfragen |

## Nächste Schritte

* Gehe zu [Kern-Konzepte](../concepts/README.md), um das mentale Modell zu festigen.
* Schau dir die komplette [Node-Referenz](../nodes/README.md) an.
* Stöbere in [Rezepten](../recipes/README.md) für mehr Beispiele.
