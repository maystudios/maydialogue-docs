---
description: Einen GameplayTag-Event feuern und externe Systeme aus dem Dialog heraus anstoßen.
---

# Fire Event

Broadcastet einen `FGameplayTag` als Ereignis an alle Listener. Der Dialog läuft sofort weiter — kein Warten, kein Handshake. Ideal um Spielsysteme außerhalb des Dialogs zu synchronisieren.

## Wann nutzen

- **Monster-Encounter auslösen** — NPC sagt "Es kommt!", `Story.Dialog.MonsterRevealed` feuert, AI-System startet das Encounter.
- **Cinematic triggern** — Dialog erreicht einen wichtigen Punkt, ein separates Sequencer-System hört auf das Event.
- **Quest-System benachrichtigen** — Spieler nimmt einen Auftrag an, `Quest.Accepted.FindKey` startet die Quest-Logik.
- **Sound-Designer-Hook** — Audio-Blueprint hört auf `Dialog.Moment.Climax` und startet einen Musik-Stinger.

---

> 📸 **Bild-Platzhalter:** `fire-event-node.png` — Node "Fire Event" im MayDialogue-Graphen.
> *Setup:* Node allein sichtbar, Title-Bar "Fire Event" (Kategorie-Farbe: gelb/Daten). Subtitle zeigt: `EventTag = Story.Dialog.MonsterRevealed`. Input- und Output-Pin sichtbar.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `EventTag` | `FGameplayTag` | Der GameplayTag der als Event gebroadcastet wird. |

---

> 📸 **Bild-Platzhalter:** `fire-event-details.png` — Details-Panel des Nodes.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `EventTag = Quest.Accepted.FindKey`. Tag-Picker offen mit Hierarchie-Vorschau.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn das Feuern des Events der **inhaltlich bedeutende Moment** im Graph ist (der Dialog-Schritt repräsentiert den Wendepunkt), nimm den Action-Node — er ist klar sichtbar und debuggbar. Wenn das Event nur ein stiller Begleiteffekt einer SayLine ist (z.B. "jedes Mal wenn dieser Satz gespielt wird"), hänge es als SideEffect-Pill an.

---

## Beispiel: Monster-Encounter synchronisieren

```text
[SayLine: Begleiterin "Hörst du das? Es kommt von oben."]
  │
  ▼
[FireEvent: EventTag=Story.Dialog.MonsterRevealed]
  │
  ▼
[SayLine: Begleiterin "Wir sollten verschwinden!"]
  │
  ▼
[Exit]
```

Ein AI-Blueprint subscribt auf `OnDialogueEventFired` und reagiert auf `Story.Dialog.MonsterRevealed`.

> 📸 **Bild-Platzhalter:** `fire-event-example-graph.png` — Graphausschnitt des obigen Beispiels.
> *Setup:* Vier Nodes: SayLine (Begleiterin) → FireEvent (Story.Dialog.MonsterRevealed im Subtitle) → SayLine (Begleiterin) → Exit. Alle Pins verbunden.

---

## Fallstricke

{% hint style="info" %}
Events sind **Fire-and-Forget** — kein Return-Wert, kein Bestätigungs-Handshake. Der Dialog wartet nicht darauf, dass ein Listener reagiert hat. Wenn du auf ein Ergebnis warten willst, nutze einen `Wait`-Node in Kombination mit einem externen System das nach Bearbeitung ein Gegenereignis zurückschickt.
{% endhint %}

- `OnDialogueEventFired` am MayDialogue-Subsystem ist der generische Listener-Punkt — beliebige externe Systeme können sich ohne Dialog-Code-Änderung einhängen.
- `EventTag` sollte einem definierten Tag aus dem Projekt-Tag-Tree entsprechen. Ein nicht registrierter Tag wirft keine Fehler, führt aber zu totem Code.
- Wait-Nodes mit passendem `WaitEventTag` werden durch diesen FireEvent aufgeweckt — nützlich für Dialog-interne Sync-Punkte.
