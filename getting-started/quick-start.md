---
description: Dialog-Asset anlegen, NPC ins Level, testen — in fünf Minuten.
---

# Quick Start

Am Ende dieser Anleitung steht ein spielbarer NPC-Dialog im Level: Ein Wächter stellt eine Frage, der Spieler wählt aus zwei Antworten, jede führt zu einer anderen Reaktion. Kein UMG-Setup, kein Audio-Handling, kein Input-Gefrickel — das Plugin erledigt das alles.

Voraussetzung: Plugin installiert (siehe [Installation](installation.md)).

---

## Schritt 1 — Dialog-Asset anlegen

1. Content Browser öffnen.
2. Rechtsklick in `Content/Dialogues/` (Ordner ggf. anlegen).
3. **Miscellaneous → MayDialogue Asset** wählen.
4. Asset benennen: `DA_Greeting_Simple`.
5. Doppelklick öffnet den Graph-Editor.

Du siehst einen leeren Graph mit einem **Entry-Node** (grüne Kapsel). Der Entry-Node ist immer vorhanden und immer der Startpunkt des Dialogs.

> 📸 **Bild-Platzhalter:** `quickstart-01-new-asset.png` — Content Browser mit dem neuen Asset `DA_Greeting_Simple`.
> *Setup:* Content Browser, Ordner `Content/Dialogues/`, das neue Asset mit dem MayDialogue-Icon sichtbar. Roter Pfeil auf das Asset.

> 📸 **Bild-Platzhalter:** `quickstart-02-empty-graph.png` — Leerer MayDialogue-Graph mit Entry-Node.
> *Setup:* Graph-Editor geöffnet mit dem Asset `DA_Greeting_Simple`. Sichtbar: nur der Entry-Node (grüne kompakte Kapsel) in der Mitte des Graphen, sonst leere Canvas. Speakers-Panel rechts leer, Outline-Panel links leer.

---

## Schritt 2 — Sprecher definieren

Im **Speakers-Panel** (Seitenleiste des Asset-Editors):

**Sprecher 1 — Wächter:**
1. **Add Speaker** klicken.
2. Tag: `Dialogue.Speaker.Guard`.
3. DisplayName: `Wächter`.
4. NodeColor: Dunkelrot.

**Sprecher 2 — Spieler:**
1. **Add Speaker** klicken.
2. Tag: `Dialogue.Speaker.Player`.
3. DisplayName: `Du`.
4. NodeColor: Grau.

> 📸 **Bild-Platzhalter:** `quickstart-03-speakers-panel.png` — Speakers-Panel mit zwei konfigurierten Sprechern.
> *Setup:* Das Speakers-Panel rechts im Asset-Editor. Zwei Einträge sichtbar: "Wächter" mit dunkelrotem Farb-Chip und Tag `Dialogue.Speaker.Guard`; "Du" mit grauem Farb-Chip und Tag `Dialogue.Speaker.Player`. Beide Einträge vollständig ausgeklappt, alle Felder sichtbar.

---

## Schritt 3 — Erste SayLine

1. Rechtsklick im Graph → **Say Line** wählen.
2. Den neuen Node im Details-Panel (oder per Doppelklick im Node) konfigurieren:
   * `SpeakerTag`: `Dialogue.Speaker.Guard`
   * `DialogueText`: `Halt! Wer bist du?`
3. Entry-Output-Pin mit dem Input-Pin der SayLine verbinden.

Die Title-Bar des Nodes nimmt automatisch die Farbe des gewählten Sprechers an (dunkelrot).

> 📸 **Bild-Platzhalter:** `quickstart-04-first-sayline.png` — Graph mit Entry-Node und erster SayLine verbunden.
> *Setup:* Graph zeigt von links nach rechts: `Entry`-Node (grüne Kapsel), Pfeil zum `SayLine`-Node (dunkelrote Title-Bar, Text "Halt! Wer bist du?" sichtbar im Node-Body, Sprecher: Wächter). Verbindungspfeil zwischen Entry-Output-Pin und SayLine-Input-Pin sichtbar.

> 📸 **Bild-Platzhalter:** `quickstart-05-sayline-properties.png` — Details-Panel der ersten SayLine.
> *Setup:* SayLine-Node ausgewählt. Details-Panel rechts zeigt: `SpeakerTag = Dialogue.Speaker.Guard`, `DialogueText = "Halt! Wer bist du?"`, `AdvanceModeOverride = (keiner)`. Rote Pfeile auf SpeakerTag und DialogueText.

---

## Schritt 4 — PlayerChoice anlegen

1. Rechtsklick im Graph → **Player Choice** wählen.
2. SayLine-Output-Pin mit PlayerChoice-Input-Pin verbinden.
3. Im Details-Panel: `PromptText` = `Du antwortest:` (optional).
4. Im Choices-Array zwei Elemente hinzufügen:
   * Choice 0: Text `Ein Freund des Königs.`
   * Choice 1: Text `Das geht dich nichts an.`

> 📸 **Bild-Platzhalter:** `quickstart-06-playerchoice.png` — PlayerChoice-Node mit zwei Choices im Graph.
> *Setup:* Graph zeigt die Kette `Entry → SayLine → PlayerChoice`. Der PlayerChoice-Node ist breiter als die SayLine, im Body sind zwei Choice-Sub-Nodes als Pills sichtbar: "Ein Freund des Königs." und "Das geht dich nichts an.". Rechts am Node zwei Output-Pins (Pin 0 und Pin 1).

---

## Schritt 5 — Reaktions-SayLines

Für jede Choice eine eigene SayLine als Reaktion:

**SayLine A:**
* `SpeakerTag`: `Dialogue.Speaker.Guard`
* `DialogueText`: `Dann passiere in Frieden.`

**SayLine B:**
* `SpeakerTag`: `Dialogue.Speaker.Guard`
* `DialogueText`: `Dann verzieh dich!`

Verbinden:
* Output-Pin 0 des PlayerChoice → SayLine A Input-Pin.
* Output-Pin 1 des PlayerChoice → SayLine B Input-Pin.

> 📸 **Bild-Platzhalter:** `quickstart-07-reactions.png` — Beide Reaktions-SayLines im Graph mit Verbindungen vom PlayerChoice.
> *Setup:* Graph zeigt `PlayerChoice` mit zwei abgehenden Pfeilen: Output-Pin 0 → SayLine A "Dann passiere in Frieden." (dunkelrot); Output-Pin 1 → SayLine B "Dann verzieh dich!" (dunkelrot). Beide SayLines rechts vom PlayerChoice, die Verbindungslinien deutlich sichtbar beschriftet mit "0" und "1".

---

## Schritt 6 — Exit

1. Rechtsklick im Graph → **Exit** wählen.
2. SayLine A Output-Pin → Exit Input-Pin verbinden.
3. SayLine B Output-Pin → Exit Input-Pin verbinden (gleicher Exit-Node).

Dein Graph sieht jetzt so aus:

```
[Entry] → [SayLine: "Halt!"] → [PlayerChoice]
                                 ├─ Pin 0 → [SayLine: "Passiere"] ─┐
                                 └─ Pin 1 → [SayLine: "Verzieh"]  ─┤
                                                                  [Exit]
```

> 📸 **Bild-Platzhalter:** `quickstart-08-final-graph.png` — Fertiger Graph mit allen Nodes und Verbindungen.
> *Setup:* Übersichtsfoto des gesamten Graphen `DA_Greeting_Simple`. Von links nach rechts: `Entry` (grüne Kapsel) → `SayLine "Halt! Wer bist du?"` (dunkelrot) → `PlayerChoice` (breit, 2 Output-Pins) → oben `SayLine "Dann passiere"` (dunkelrot) und unten `SayLine "Dann verzieh dich!"` (dunkelrot) → beide Pfeile treffen sich am gemeinsamen `Exit` (rote Kapsel). Layout horizontal, alle Verbindungspfeile klar sichtbar.

---

## Schritt 7 — Compile

**Toolbar → Compile** klicken.

Falls der Validator Fehler meldet, behebe sie im **Compiler Results**-Panel:

| Fehler | Ursache | Lösung |
| --- | --- | --- |
| Unverbundener Output-Pin | Ein Node hat keinen Ausgang | Output-Pin an Exit oder nächsten Node hängen |
| Sprecher-Tag leer | SayLine ohne Speaker | Im Details-Panel SpeakerTag setzen |
| Leere SayLine | Kein Text | Text eintragen |

---

## Schritt 8 — NPC ins Level setzen

1. Level öffnen.
2. Beliebigen Actor als Wächter-Platzhalter ins Level setzen (z.B. einen Character Blueprint oder einen StaticMesh-Actor).
3. Actor in der Details-Panel: **Add Component → MayDialogue Participant**.
4. Komponente konfigurieren:
   * `ParticipantTag`: `Dialogue.Speaker.Guard`
   * `DisplayName`: `Wächter`
   * `DefaultDialogue`: `DA_Greeting_Simple`

{% hint style="info" %}
Dein **Spieler-Pawn** braucht ebenfalls eine `MayDialogueParticipant`-Komponente mit `ParticipantTag = Dialogue.Speaker.Player`. Auch wenn der Spieler im Quick Start keine eigenen SayLines hat, muss das Plugin wissen, wer der Instigator ist.
{% endhint %}

> 📸 **Bild-Platzhalter:** `quickstart-09-participant-component.png` — Details-Panel des Wächter-Actors mit MayDialogueParticipant-Komponente.
> *Setup:* Details-Panel eines Level-Actors. In der Komponenten-Liste ist `MayDialogueParticipant` sichtbar (ausgewählt). Darunter die Properties: `ParticipantTag = Dialogue.Speaker.Guard`, `DisplayName = Wächter`, `DefaultDialogue = DA_Greeting_Simple`. Roter Pfeil auf `DefaultDialogue`.

---

## Schritt 9 — Dialog auslösen (Blueprint)

Im Blueprint-Graph deines Trigger-Actors oder deiner Spieler-Logik:

**Variante A — direkt über den Participant:**

1. Referenz zum Wächter-Actor holen.
2. **Get Component by Class: MayDialogueParticipant** aufrufen.
3. Auf dem Participant: **Start Default Dialogue** aufrufen.
4. `Other`-Parameter: Referenz zur Spieler-Participant-Komponente.

**Variante B — über die Library-Funktion:**

```
MayDialogueLibrary → StartDialogue
  WorldContext = Self
  Asset        = DA_Greeting_Simple
  Instigator   = PlayerPawn
  Target       = GuardActor
```

> 📸 **Bild-Platzhalter:** `quickstart-10-blueprint-trigger.png` — Blueprint-Graph eines Trigger-Actors mit dem StartDialogue-Aufruf.
> *Setup:* BP-Graph eines Box-Trigger-Actors. Event `OnComponentBeginOverlap` → `Get Component by Class (MayDialogueParticipant)` auf dem Wächter-Actor → `Start Default Dialogue` mit `Other` = Spieler-Participant-Referenz. Alle Pins beschriftet, Ausführungspfeile sichtbar.

{% hint style="info" %}
**Variante für C++:**

```cpp
UMayDialogueParticipantComponent* GuardParticipant =
    Guard->FindComponentByClass<UMayDialogueParticipantComponent>();
if (GuardParticipant)
{
    GuardParticipant->StartDefaultDialogue(PlayerParticipant);
}
```
{% endhint %}

---

## Schritt 10 — Testen

**PIE starten**, zum Wächter gehen und den Trigger auslösen.

Was du siehst:
* Ein Widget erscheint im Viewport (Slate-Debug-Widget, solange noch kein UMG-Widget gesetzt ist).
* Der Wächter spricht: *"Halt! Wer bist du?"*.
* Zwei Choice-Buttons erscheinen.
* Klick auf eine Choice führt zur passenden Reaktion und der Dialog endet.

> 📸 **Bild-Platzhalter:** `quickstart-11-ingame.png` — In-Game-Screenshot des laufenden Dialogs im PIE.
> *Setup:* PIE aktiv. Im Viewport ist das Slate-Debug-Widget sichtbar: oben der Sprecher-Name "Wächter", darunter der Text "Halt! Wer bist du?", unten zwei Choice-Buttons "Ein Freund des Königs." und "Das geht dich nichts an.". Der Wächter-Actor im Hintergrund sichtbar.

{% hint style="success" %}
Du hast einen spielbaren Dialog ohne UMG-Setup, ohne Audio-Konfiguration und ohne Input-Handling-Code.
{% endhint %}

---

## Fehlerbehebung

<details>
<summary>Kein Widget erscheint beim Dialog-Start</summary>

Prüfe in **Edit → Project Settings → Plugins → MayDialogue**, ob `bUseSlateDialogueWidget` aktiviert ist. Standardmäßig erscheint das Slate-Debug-Widget auch ohne gesetztes UMG-Widget.
</details>

<details>
<summary>Dialog startet gar nicht</summary>

Prüfe:
* Hat das Asset einen Entry-Node und wurde es compiliert (Toolbar → Compile)?
* Hat der Wächter-Actor eine `MayDialogueParticipant`-Komponente mit dem richtigen Tag?
* Hat der Spieler-Pawn ebenfalls eine Participant-Komponente?
* Der Output-Log zeigt Warnungen, wenn etwas fehlt — dort nach `MayDialogue` suchen.
</details>

<details>
<summary>Die Frage erscheint, aber keine Choice-Buttons</summary>

Das passiert, wenn der PlayerChoice-Node keine verbundenen Output-Pins hat oder alle Choices aufgrund von Requirements verborgen sind. Öffne das Asset, prüfe die Choices-Liste im PlayerChoice-Node.
</details>

---

## Was als Nächstes?

* Variablen, Branching und GAS-Requirements kennenlernen → [Walkthrough](first-dialogue.md)
* Eigenes UMG-Widget statt Slate-Debug-Widget → [UI-Architektur](../ui/umg-architecture.md)
* Audio hinzufügen → [Audio-System](../audio/README.md)
* Choices an GAS-Attribute knüpfen → [GAS-Integration](../gas/README.md)
