---
description: Einen LooseGameplayTag auf dem Spieler oder einem NPC setzen — leichtgewichtig, kein GE-Overhead.
---

# Add Tag

Fügt einen `FGameplayTag` direkt per `AddLooseGameplayTag` zu einem `AbilitySystemComponent` hinzu. Kein GameplayEffect-Overhead — ideal für Dialog-State-Flags, die über den Dialog hinaus bestehen bleiben.

## Wann nutzen

- **Dialog-State-Flag am NPC** — Wächter "weiß ein Geheimnis": `Story.Guard.KnowsSecret` am Wächter-ASC.
- **Spieler hat etwas erlebt** — `Story.Player.HeardProphecy` sichert, dass dieser Dialog-Ast nur einmal zugänglich ist.
- **Schalter für andere Systeme** — Quest-System, AI-StateTree oder andere Dialoge prüfen diesen Tag per `HasTag`.
- **Freischalt-Mechanismus** — NPC vertraut dem Spieler erst wenn `Story.Merchant.TrustGained` gesetzt ist.

---

> 📸 **Bild-Platzhalter:** `add-tag-node.png` — Node "Add Tag" im MayDialogue-Graphen.
> *Setup:* Node allein, Title-Bar "Add Tag" (Kategorie-Farbe: lila/GAS). Subtitle zeigt: `Story.Guard.KnowsSecret → Target`. Input- und Output-Pin sichtbar.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `TagToAdd` | `FGameplayTag` | Der Tag der hinzugefügt wird. |
| `bAddToInstigator` | `bool` | `true` = Tag geht auf den Spieler-ASC. `false` = Tag geht auf den Ziel-NPC-ASC. Default: `true`. |

---

> 📸 **Bild-Platzhalter:** `add-tag-details.png` — Details-Panel mit NPC-Target-Setup.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `TagToAdd = Story.Guard.KnowsSecret`, `bAddToInstigator = false`.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn das Setzen des Tags der **inhaltliche Hauptpunkt** des Schrittes ist (der Graph repräsentiert den Moment in dem der NPC etwas erfährt), nimm den Action-Node. Wenn der Tag nur ein stilles Begleit-Tracking einer SayLine ist (jedes Mal wenn der Spieler diese Zeile hört), hänge ihn als SideEffect-Pill an die SayLine.

---

## LooseTag vs. GameplayEffect-Tag

| | LooseTag (dieser Node) | GameplayEffect-Tag |
|---|---|---|
| Persistenz | Bis explizit entfernt ([Remove Tag](remove-tag.md)) | Bis GE endet oder endet durch Stack |
| Overhead | Sehr gering | GE-Spec + Modifier-Berechnung |
| Replikation | Eingeschränkt (Minimal-Modus) | Voll via GAS-Replikation |
| Typischer Use-Case | Dialog-Flags, Story-State | Gameplay-Zustände mit Lifetime |

Für Story-Flags wie "hat mich gegrüßt", "kennt mein Geheimnis" ist LooseTag die richtige Wahl.

---

## Beispiel: NPC-Wissens-Flag setzen

```text
[PlayerChoice: "Ich erzähle dir ein Geheimnis..."]
  │
  ▼
[AddTag: TagToAdd=Story.Guard.KnowsSecret, bAddToInstigator=false]
  │
  ▼
[SayLine: NPC "Das darf niemand erfahren..."]
  │
  ▼
[Exit]
```

> 📸 **Bild-Platzhalter:** `add-tag-example-graph.png` — Graphausschnitt des obigen Beispiels.
> *Setup:* Vier Nodes: PlayerChoice → AddTag (Story.Guard.KnowsSecret im Subtitle, "→ Target") → SayLine (NPC) → Exit. Alle Pins verbunden.

---

## Fallstricke

{% hint style="info" %}
`AddLooseGameplayTag` ist **idempotent** — mehrmaliges Hinzufügen desselben Tags hat keinen Stack-Effekt. Der Tag ist entweder vorhanden oder nicht. Wenn du einen Stack-basierten Zähler brauchst, nutze stattdessen ein Int-Attribut via `Apply Effect`.
{% endhint %}

- Der Node erfordert das **MayDialogueGAS-Modul** — ohne dieses Modul ist der Node nicht verfügbar.
- Ziel (Instigator oder Target-NPC) muss einen `UAbilitySystemComponent` haben, sonst No-Op + Log-Warning.
- LooseTags werden bei Replikation auf `Minimal`-Modus nicht automatisch auf andere Clients repliziert — für Multiplayer-State lieber `Apply Effect` mit Tag-Grant verwenden.
- Partner-Node: [Remove Tag](remove-tag.md).
