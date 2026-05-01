---
description: Variablen, Choice-Requirements und SideEffect-Aktionen — ein vollständiger Dialog.
---

# Walkthrough — Ein vollständiger Dialog

Der [Quick Start](quick-start.md) hat gezeigt, wie du in fünf Minuten einen spielbaren Dialog baust. Dieser Walkthrough geht eine Stufe weiter. Du baust ein realistisches NPC-Gespräch, das Variablen nutzt, Choices an GAS-Tags und Attribute knüpft und SideEffect-Aktionen auslöst.

{% hint style="warning" %}
**Voraussetzungen für Schritte 6–7 (GAS-Attribut und ApplyEffect):**

- Das **Gameplay Ability System** (GAS) muss in deinem Projekt aktiviert sein.
- Du benötigst ein eigenes `UAttributeSet` mit dem Attribut `Reputation.Guards` auf dem Spieler-ASC.
- Du benötigst einen GameplayEffect `GE_GuardTrust`, der dieses Attribut erhöht.

**Kein GAS in deinem Projekt?** Überspringe Schritt 6 (Requirement auf Choice 1) und Schritt 7 (ApplyEffect) und ersetze sie durch einfache Variable-Checks bzw. SetVariable-SideEffects aus dem Dialogue-Scope. Eine nicht-GAS-Variante findest du im Abschnitt [Optional: Variablen-Variante ohne GAS](#optional-variablen-variante-ohne-gas) am Ende dieses Walkthroughs.
{% endhint %}

## Das Szenario

Du triffst einen Wächter vor einer Festung. Drei Dinge bestimmen, was passiert:

1. **Hast du ihn schon einmal getroffen?** — eine Participant-Variable `HasMet` (Bool).
2. **Hast du das Passwort gehört?** — ein GameplayTag `Story.Secret.HeardPassword` auf dem Spieler.
3. **Wie ist deine Reputation?** — ein GAS-Attribut `Reputation.Guards` auf dem Spieler.

---

## Schritt 1 — Neues Dialog-Asset anlegen

1. Content Browser → `Content/Dialogues/` → Rechtsklick → **May Dialogue → Dialogue Asset**.
2. Benennen: `DA_Gate_Guardian`.
3. Doppelklick öffnet den Graph-Editor.

### Sprecher anlegen

Im **Speakers-Panel**:

| Tag | DisplayName | Farbe |
| --- | --- | --- |
| `Dialogue.Speaker.Guard` | Wächter | Dunkelrot |
| `Dialogue.Speaker.Player` | Du | Grau |

### Participant-Variable anlegen

Im **Variables-Panel** (Seitenleiste):

1. **Add Variable** klicken.
2. Name: `HasMet`, Typ: `Bool`, Scope: `Participant`, Default: `false`.

Participant-Scope bedeutet: die Variable gehört dem Wächter-Actor und bleibt zwischen Dialog-Starts erhalten (solange der Actor im Level lebt).

> 📸 **Bild-Platzhalter:** `walkthrough-01-variables-panel.png` — Variables-Panel mit der HasMet-Variable.
> *Setup:* Variables-Panel im Asset-Editor, ein Eintrag sichtbar: `HasMet`, Typ `Bool`, Scope `Participant`, Default `false`. Roter Pfeil auf den Scope-Dropdown.

---

## Schritt 2 — Begrüßung mit Branch

Der Einstieg fragt: Hat der Spieler den Wächter schon getroffen?

1. Rechtsklick im Graph → **Branch** platzieren.
2. Entry-Output-Pin → Branch-Input-Pin verbinden.
3. Am Branch-Node einen Requirement-Sub-Node hinzufügen:
   * Sub-Node-Typ: **CheckParticipantVariable**
   * `VariableName`: `HasMet`
   * `ExpectedValue`: `true`
   * `CheckTarget`: `Target` (der Wächter ist das Target des Dialogs)

**True-Ausgang** → **SayLine** "Du bist also wieder hier. Was willst du?"
**False-Ausgang** → **SayLine** "Halt! Wer bist du?"

> 📸 **Bild-Platzhalter:** `walkthrough-02-branch-node.png` — Branch-Node mit CheckParticipantVariable-Sub-Node im Graph.
> *Setup:* Graph zeigt: `Entry` → `Branch` (Diamant-Form mit True-, False- und Fallback-Pin). Am Branch-Node ist im Body der Sub-Node "CheckParticipantVariable: HasMet == true" als Pill sichtbar. Vom True-Pin geht ein Pfeil zur SayLine "Du bist also wieder hier." (dunkelrot), vom False-Pin zur SayLine "Halt! Wer bist du?" (dunkelrot). Fallback-Pin nicht verbunden (zeigt leeren Pin).

> 📸 **Bild-Platzhalter:** `walkthrough-03-branch-subnodes.png` — Details-Panel des CheckParticipantVariable-Requirements.
> *Setup:* Branch-Node ausgewählt, im Details-Panel der eingebettete Requirement sichtbar: `VariableName = HasMet`, `ExpectedValue = true`, `CheckTarget = Target`. Roter Pfeil auf `CheckTarget`.

---

## Schritt 3 — HasMet per SideEffect setzen

Jede Begrüßungs-SayLine (True- und False-Pfad) soll beim Betreten `HasMet = true` setzen — egal welche Variante gespielt wird.

**Auf beiden Begrüßungs-SayLines** jeweils:

1. SayLine auswählen.
2. Im Sub-Nodes-Bereich: **Add SideEffect → SetVariable**.
3. `VariableName`: `HasMet`, `NewValue`: `true`, `Scope`: `Participant`, `Target`: Wächter-Participant.

Der SideEffect erscheint als Pill im Node-Body der SayLine — sichtbar, aber nicht aufdringlich.

> 📸 **Bild-Platzhalter:** `walkthrough-04-sideeffect-pill.png` — SayLine "Halt! Wer bist du?" mit SideEffect-Pill im Node-Body.
> *Setup:* SayLine-Node "Halt! Wer bist du?" im Graph. Im unteren Bereich des Node-Bodys ist eine Pill sichtbar: kleines Icon (Stift oder Variable-Symbol) + Text "SetVariable: HasMet = true". Roter Pfeil auf die Pill.

---

## Schritt 4 — PlayerChoice mit drei Optionen

Beide Begrüßungs-SayLines münden in denselben **PlayerChoice**-Node:

1. True-SayLine-Output → PlayerChoice-Input verbinden.
2. False-SayLine-Output → PlayerChoice-Input verbinden.
3. Im Choices-Array drei Elemente anlegen:
   * Choice 0: `Ich kenne das Passwort: Elenderion.`
   * Choice 1: `Ich bin ein Freund des Königs.`
   * Choice 2: `Das geht dich nichts an.`

> 📸 **Bild-Platzhalter:** `walkthrough-05-playerchoice-overview.png` — PlayerChoice-Node mit drei Choices und zwei eingehenden Pfeilen.
> *Setup:* Graph zeigt die SayLine "Halt!" (links oben) und die SayLine "Du bist also wieder hier." (links unten), beide mit Pfeilen zum PlayerChoice-Node (mitte-rechts). Im PlayerChoice-Body drei Choice-Einträge als Pills: "Ich kenne das Passwort…", "Ich bin ein Freund…", "Das geht dich nichts an.". Rechts drei Output-Pins (0, 1, 2).

---

## Schritt 5 — Requirement auf Choice 0 (Passwort)

Choice 0 soll nur erscheinen, wenn der Spieler den Tag `Story.Secret.HeardPassword` hat.

1. Choice 0 auswählen.
2. **Add Requirement → HasTag**.
3. `RequiredTag`: `Story.Secret.HeardPassword`.
4. `CheckOnInstigator`: `true` (der Spieler ist der Instigator).
5. `FailureResult`: `FailedAndHidden` — die Choice verschwindet komplett, wenn die Bedingung nicht erfüllt ist.

> 📸 **Bild-Platzhalter:** `walkthrough-06-requirement-hastag.png` — Details-Panel der HasTag-Requirement auf Choice 0.
> *Setup:* Choice 0 des PlayerChoice-Nodes ausgewählt. Details-Panel zeigt: `RequiredTag = Story.Secret.HeardPassword`, `CheckOnInstigator = true`, `FailureResult = FailedAndHidden`. Roter Pfeil auf `FailureResult`.

Das ergibt das klassische RPG-Muster: Die Passwort-Option siehst du nur, wenn du das Passwort tatsächlich gehört hast.

---

## Schritt 6 — Requirement auf Choice 1 (Reputation)

Choice 1 soll sichtbar, aber nicht wählbar sein, wenn die Reputation zu niedrig ist.

1. Choice 1 auswählen.
2. **Add Requirement → CheckAttribute**.
3. `Attribute`: `Reputation.Guards`.
4. `ComparisonOp`: `>=`.
5. `ComparisonValue`: `50`.
6. `CheckOnInstigator`: `true`.
7. `FailureResult`: `FailedButVisible` — die Choice erscheint gegraut, ist aber nicht wählbar.
8. `UnavailableReason`: `Deine Reputation bei den Wachen ist zu niedrig.` (erscheint als Tooltip).

> 📸 **Bild-Platzhalter:** `walkthrough-07-requirement-attribute.png` — Details-Panel des CheckAttribute-Requirements auf Choice 1.
> *Setup:* Choice 1 des PlayerChoice-Nodes ausgewählt. Details-Panel zeigt: `Attribute = Reputation.Guards`, `ComparisonOp = >=`, `ComparisonValue = 50`, `CheckOnInstigator = true`, `FailureResult = FailedButVisible`, `UnavailableReason = "Deine Reputation bei den Wachen ist zu niedrig."`. Roter Pfeil auf `FailedButVisible`.

> 📸 **Bild-Platzhalter:** `walkthrough-08-ingame-choices.png` — In-Game-Screenshot des PlayerChoice-Widgets mit einer grauen Choice.
> *Setup:* PIE läuft. Widget zeigt drei Choices: Choice 0 fehlt (weil HasHeardPassword nicht gesetzt), Choice 1 ist sichtbar aber ausgegraut mit Tooltip "Deine Reputation bei den Wachen ist zu niedrig.", Choice 2 ist normal klickbar "Das geht dich nichts an.". Roter Pfeil auf die graue Choice 1.

---

## Schritt 7 — Konsequenzen: Choice 0 mit ApplyEffect

**Pfad für Choice 0 (Passwort):**

Platziere nach Output-Pin 0 des PlayerChoice:

1. **Action-Node: Apply Effect**
   * `EffectClass`: `GE_GuardTrust` (dein Projekt-eigener GameplayEffect, der Reputation erhöht)
   * `EffectLevel`: `1.0`
   * `ApplyToInstigator`: `true`
2. **SayLine**: `Dann passiere in Frieden.` (Sprecher: Wächter)
3. **Exit** (Status: Completed)

Verbindungen: `PlayerChoice OutputPin[0]` → `ApplyEffect` → `SayLine "Passiere"` → `Exit`.

> 📸 **Bild-Platzhalter:** `walkthrough-09-choice0-path.png` — Pfad von Choice 0 mit ApplyEffect-Node im Graph.
> *Setup:* Rechts vom PlayerChoice, oben: Pfeil von Output-Pin 0 → `ApplyEffect`-Node (Body zeigt "GE_GuardTrust, Level 1.0, Instigator") → `SayLine "Dann passiere in Frieden."` (dunkelrot) → `Exit` (rote Kapsel, Status: Completed). Alle Verbindungspfeile sichtbar.

---

## Schritt 8 — Konsequenzen: Choice 1 und Choice 2

**Pfad für Choice 1 (König):**

`PlayerChoice OutputPin[1]` → `SayLine "Dann passiere, Freund."` → `Exit (Completed)`.

**Pfad für Choice 2 (frech):**

Platziere:
1. **Action-Node: Add Tag** — `Tag`: `Story.Guard.Hostile`, `AddToInstigator`: `false` (Tag wird auf den Wächter gesetzt)
2. **Action-Node: Camera Shake** — dein Projekt-eigener `UCameraShakeBase`, `Scale: 1.5`
3. **SayLine** `Dann verzieh dich von hier!` (Sprecher: Wächter)
4. **Exit** (Status: Failed)

Verbindungen: `PlayerChoice OutputPin[2]` → `AddTag` → `CameraShake` → `SayLine "Verzieh dich"` → `Exit (Failed)`.

> 📸 **Bild-Platzhalter:** `walkthrough-10-choice2-path.png` — Pfad von Choice 2 mit AddTag- und CameraShake-Nodes.
> *Setup:* Rechts vom PlayerChoice, unten: Pfeil von Output-Pin 2 → `AddTag`-Node (Body: "Story.Guard.Hostile → Target") → `CameraShake`-Node (Body: "Scale 1.5") → `SayLine "Dann verzieh dich!"` (dunkelrot) → `Exit (Failed)` (rote Kapsel, Status: Failed). Alle Pfeile beschriftet.

---

## Schritt 9 — RandomLine für wiederholte Besuche

Die SayLine "Du bist also wieder hier." wird nach dem dritten Besuch langweilig. Ersetze sie durch einen **Random Line**-Node:

1. SayLine "Du bist also wieder hier." löschen.
2. Rechtsklick → **Random Line** platzieren.
3. Im Details-Panel drei Lines eintragen:
   * `Du bist also wieder hier. Was willst du?`
   * `Zurück? Hoffe, du hast einen guten Grund.`
   * `Schon wieder. Und heute?`
4. `bRememberSelection`: `true` — verhindert zwei identische Zeilen hintereinander.
5. True-Pfad des Branch → RandomLine-Input verbinden.
6. RandomLine-Output → PlayerChoice-Input verbinden (wie zuvor die SayLine).

> 📸 **Bild-Platzhalter:** `walkthrough-11-random-line.png` — RandomLine-Node im Graph mit Würfel-Icon und drei Lines im Body.
> *Setup:* RandomLine-Node (Würfel-Icon in der Title-Bar). Im Node-Body drei Lines als Pills sichtbar: "Du bist also wieder hier…", "Zurück?…", "Schon wieder…". Details-Panel zeigt `bRememberSelection = true`. Pfeil eingehend vom True-Pin des Branch, Pfeil ausgehend zum PlayerChoice.

---

## Schritt 10 — Compile und Test

1. **Toolbar → Compile** klicken. Keine Fehler im Compiler Results Panel?
2. PIE starten.

**Testmatrix:**

| Zustand | Erwartetes Verhalten |
| --- | --- |
| Erster Besuch, kein Passwort, Reputation < 50 | Begrüßung "Halt!"; Choice 0 fehlt; Choice 1 gegraut; nur Choice 2 wählbar |
| Erster Besuch, Passwort gehört, Reputation >= 50 | Alle drei Choices sichtbar und wählbar |
| Zweiter Besuch | RandomLine-Begrüßung statt "Halt!" |
| Choice 0 gewählt | ApplyEffect erhöht Reputation; Wächter verabschiedet freundlich |
| Choice 2 gewählt | `Story.Guard.Hostile`-Tag gesetzt; CameraShake; Wächter wirft raus |

> 📸 **Bild-Platzhalter:** `walkthrough-12-full-graph-overview.png` — Übersichtsfoto des vollständigen Graphen DA_Gate_Guardian.
> *Setup:* Gesamter Graph in der Vogelperspektive (ausgezoomt). Sichtbar von links nach rechts: `Entry` → `Branch` → True-Pfad (RandomLine → PlayerChoice) und False-Pfad (SayLine "Halt!" → PlayerChoice). Vom PlayerChoice drei Ausgänge nach rechts: oben Choice-0-Pfad (ApplyEffect → SayLine → Exit Completed), mitte Choice-1-Pfad (SayLine → Exit Completed), unten Choice-2-Pfad (AddTag → CameraShake → SayLine → Exit Failed). SideEffect-Pills auf den Begrüßungs-Nodes sichtbar.

---

## Was du gebaut hast

| Feature | Eingesetzt in |
| --- | --- |
| Participant-Variable | `HasMet` — merkt sich den ersten Besuch |
| Branch-Node + Requirement | Begrüßungs-Verzweigung je nach `HasMet` |
| SideEffect → SetVariable | `HasMet` auf `true` setzen beim ersten Betreten |
| PlayerChoice mit Requirements | GAS-Tag- und Attribut-gesteuerte Optionen |
| Action-Node: ApplyEffect | Reputation erhöhen nach Passwort |
| Action-Node: AddTag | Wächter als `Story.Guard.Hostile` markieren |
| Action-Node: CameraShake | Dramatischer Moment beim Rauswurf |
| RandomLine | Abwechslungsreiche Begrüßung bei Wiederholung |

---

## Nächste Schritte

* [Kern-Konzepte](../concepts/README.md) — das mentale Modell verstehen
* [Node-Referenz](../nodes/README.md) — alle Nodes im Detail
* [Rezepte](../recipes/README.md) — weitere Beispiele für häufige Patterns

{% hint style="info" %}
**Eigene Requirement-Typen bauen:** Lege eine Blueprint-Klasse mit Parent `UMayDialogueRequirement` an. In der `IsRequirementSatisfied`-Funktion gibst du `Passed`, `FailedButVisible` oder `FailedAndHidden` zurück. Der neue Typ erscheint sofort in der Sub-Node-Palette. Siehe [Eigene Requirements](../extension/custom-requirements.md).
{% endhint %}

---

## Optional: Variablen-Variante ohne GAS

Falls GAS in deinem Projekt nicht aktiv ist, kannst du Schritte 6 und 7 durch variable-basierte Alternativen ersetzen:

**Statt Schritt 6 (CheckAttribute-Requirement auf Choice 1):**

Lege im Variables-Panel eine Dialogue-Variable `Reputation` (Typ: `Int`, Default: `0`) an. Verwende als Requirement **CheckDialogueVariable** mit `VariableName = Reputation`, `ComparisonOp = >=`, `ComparisonValue = 50`.

**Statt Schritt 7 (ApplyEffect):**

Ersetze den Apply-Effect-Node durch einen **Set Variable**-Node: `VariableName = Reputation`, `NewValue = Reputation + 50`. Da direkte Rechenoperationen im Graph nicht verfügbar sind, kannst du hierfür einen kleinen Blueprint-SideEffect anlegen (siehe [Variablen & Scopes](../concepts/variables-scopes.md)).

Die Testmatrix und der restliche Walkthrough bleiben identisch.
