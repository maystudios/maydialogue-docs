---
description: Einen LooseGameplayTag vom Spieler oder einem NPC entfernen.
---

# Remove Tag

Entfernt einen `FGameplayTag` per `RemoveLooseGameplayTag` von einem `AbilitySystemComponent`. Greift ausschließlich auf Tags, die via `AddLooseGameplayTag` gesetzt wurden — GameplayEffect-granted Tags bleiben unberührt.

## Wann nutzen

- **Vertrauen zurückgewinnen** — NPC vergab dem Spieler: `Story.Guard.Distrusts` entfernen.
- **Questphase abschließen** — Nachdem ein Ziel erfüllt ist, Tracking-Flag entfernen.
- **Reset nach Wahl** — Spieler wechselt Seite im Story-Branch; altes Fraktions-Flag löschen.
- **Zustand-Bereinigung** — Dialog-Ende räumt temporäre Flags auf, die nur während des Gesprächs galten.

---

> 📸 **Bild-Platzhalter:** `remove-tag-node.png` — Node "Remove Tag" im MayDialogue-Graphen.
> *Setup:* Node allein, Title-Bar "Remove Tag" (Kategorie-Farbe: lila/GAS). Subtitle zeigt: `Story.Guard.Distrusts ← Target`. Input- und Output-Pin sichtbar.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `TagToRemove` | `FGameplayTag` | Der Tag der entfernt wird. |
| `bRemoveFromInstigator` | `bool` | `true` = Tag wird vom Spieler-ASC entfernt. `false` = Tag wird vom Ziel-NPC-ASC entfernt. Default: `true`. |

---

> 📸 **Bild-Platzhalter:** `remove-tag-details.png` — Details-Panel mit NPC-Target-Setup.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `TagToRemove = Story.Guard.Distrusts`, `bRemoveFromInstigator = false`.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn das Entfernen des Tags der **semantische Hauptpunkt** des Schrittes ist (der Moment in dem Vertrauen zurückgewonnen wird), nimm den Action-Node. Wenn es nur ein stilles Cleanup am Ende einer SayLine ist, hänge es als SideEffect-Pill an den Exit-Node.

---

## Beispiel: Vertrauen zurückgewinnen

```text
[SayLine: Wächter "Du hast dich bewährt. Ich vertraue dir wieder."]
  │
  ▼
[RemoveTag: TagToRemove=Story.Guard.Distrusts, bRemoveFromInstigator=false]
  │
  ▼
[Exit Completed]
```

> 📸 **Bild-Platzhalter:** `remove-tag-example-graph.png` — Graphausschnitt des obigen Beispiels.
> *Setup:* Drei Nodes: SayLine (Wächter) → RemoveTag (Story.Guard.Distrusts, "← Target" im Subtitle) → Exit. Alle Pins verbunden.

---

## Fallstricke

{% hint style="info" %}
`RemoveLooseGameplayTag` ist **idempotent** — wenn der Tag gar nicht gesetzt war, passiert nichts (kein Fehler, kein Log). Sicher zu nutzen ohne vorherige HasTag-Prüfung.
{% endhint %}

{% hint style="warning" %}
Dieser Node entfernt **nur LooseTags**. Tags die durch einen `UGameplayEffect` gegraanted wurden, bleiben bestehen. Um GE-Tags zu entfernen, muss der Effect selbst entfernt werden (z.B. via `Apply Effect` mit einer `GE_Remove`-Klasse oder durch Ablauf).
{% endhint %}

- Der Node erfordert das **MayDialogueGAS-Modul** — ohne dieses Modul ist der Node nicht verfügbar.
- Ziel muss einen `UAbilitySystemComponent` haben — sonst No-Op + Log-Warning.
- Partner-Node: [Add Tag](add-tag.md).
