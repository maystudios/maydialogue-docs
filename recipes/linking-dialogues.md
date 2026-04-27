---
description: Denselben Dialog-Block in mehreren Assets nutzen – Link-Node vs. SubGraph, Entscheidungshilfe und Setup.
---

# Wiederverwendbare Dialog-Fragmente

## Szenario

Alle Händler im Spiel enden ihr Gespräch mit demselben Farewell-Block: *„Guten Weg, Reisender."* gefolgt von einer zufälligen Warnung. Statt diesen Block in jedes Asset zu kopieren, lagerst du ihn einmal aus. Bei fünf Händlern sparst du fünffache Pflege – und Bugfixes greife an einer Stelle.

## Was du lernst

- Link-Node auf ein externes Fragment-Asset zeigen lassen.
- `bReturnAfterExit` nutzen, damit der Host-Dialog nach dem Fragment weiterläuft.
- SubGraph als Alternative für interne Strukturierung.
- Wann Link, wann SubGraph.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.

## Mini-Graph

**Fragment-Asset `DA_Common_Farewell`:**

```text
[Entry]
   │
   ▼
[SayLine: "Guten Weg, Reisender."]
   │
   ▼
[RandomLine: 3 Warnungen über die Wälder]
   │
   ▼
[Exit: Completed]
```

**Händler-Asset `DA_Merchant_*`:**

```text
[... Händler-spezifisches Gespräch ...]
   │
   ▼
[Link → DA_Common_Farewell  bReturnAfterExit: true]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `linking-dialogues-graph-overview.png` — Zwei Assets nebeneinander: Fragment-Asset links, Händler-Asset rechts mit Link-Node.
> *Setup:* Content Browser und zwei Editor-Fenster nebeneinander. Links: `DA_Common_Farewell` mit Entry → SayLine → RandomLine → Exit. Rechts: `DA_Merchant_A` mit Gesprächsblock → Link-Node (Portal-Icon) → Exit. Link-Node hat `LinkedAsset = DA_Common_Farewell` im Details-Panel sichtbar.

## Schritt-für-Schritt – Link-Variante

### 1. Fragment-Asset anlegen

Neues Asset `DA_Common_Farewell`. Speaker: `Dialogue.Speaker.CurrentSpeaker` (generischer Token – see Pro-Tipp).

Graph: Entry → SayLine *„Guten Weg, Reisender."* → RandomLine mit drei Warnungen → Exit (`Completed`).

### 2. Link-Node im Händler-Dialog einbauen

Im Händler-Asset am Ende des eigentlichen Gesprächs: Vom letzten Output-Pin → **Create Node → Link**.

### 3. Link-Node konfigurieren

| Property | Wert |
|----------|------|
| `LinkedAsset` | `DA_Common_Farewell` |
| `EntryGuid` | *(leer = Standard-Entry)* |
| `bReturnAfterExit` | `true` |

### 4. Rückkehr verdrahten

Der Link-Node hat einen Output-Pin, der feuert wenn das Fragment beendet ist. Verbinde ihn mit deinem finalen Exit-Node.

> 📸 **Bild-Platzhalter:** `linking-dialogues-link-node-details.png` — Details-Panel des Link-Nodes.
> *Setup:* Link-Node ausgewählt. Details zeigt: `LinkedAsset = DA_Common_Farewell`, `EntryGuid = (leer)`, `bReturnAfterExit = true (Checkbox aktiviert)`.

{% hint style="info" %}
**Pro-Tipp Speaker-Tokens:** Statt harter Speaker-Tags im Fragment nutze `Dialogue.Speaker.CurrentSpeaker`. Im Host-Asset wird dieser Token auf den aktuell sprechenden Händler gemappt – so funktioniert das Fragment für jeden Händler ohne Anpassung.
{% endhint %}

## SubGraph-Variante (für rein interne Strukturierung)

Wenn das Fragment **nicht** in anderen Assets gebraucht wird, sondern nur den eigenen Graph aufräumen soll:

Vom letzten Output-Pin → **Create Node → SubGraph**. Benenne ihn *„Farewell Block"*. Doppelklick öffnet den Sub-Graph in einem neuen Tab mit Breadcrumb-Navigation.

```text
[... Händler-Gespräch ...]
   │
   ▼
[SubGraph: "Farewell Block"]
   │ (intern: Entry → SayLine → RandomLine → Exit)
   ▼
[Exit: Completed]
```

## Link vs. SubGraph

| Kriterium | Link | SubGraph |
|-----------|------|----------|
| Wiederverwendung über Asset-Grenzen | Ja | Nein |
| Collapsible im eigenen Asset | Nein | Ja |
| Eigenes Speakers-Scope | Ja | Nein (erbt vom Parent) |
| Bugfix greift überall | Ja (zentral) | Nein (lokal) |
| Merge-Konflikte | Weniger (getrennte Assets) | Mehr (alles in einer Datei) |

**Faustregel:** Wiederverwendung über mehrere Assets → **Link**. Nur internes Aufräumen → **SubGraph**.

## Blueprint-Triggering

Für den Aufrufer ändert sich nichts. Normaler Start-Call – das Fragment läuft in derselben Instance:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Merchant_A
   └─ ...
```

> 📸 **Bild-Platzhalter:** `linking-dialogues-bp-trigger.png` — Blueprint-Trigger am Händler.
> *Setup:* Händler-Actor Blueprint. `Event OnInteract` → `Start Dialogue (MayDialogueLibrary)`, `Asset = DA_Merchant_A`.

## Variation / Weiter gehen

- Vor dem Link-Node eine Variable setzen (`SetVariable`), die das Fragment liest – Parameter übergeben ohne neue Assets. Siehe [GAS-getriebener Dialog](gas-driven-dialogue.md).
- Fragment abhängig von Spieler-Tags variieren: Branch im Fragment-Asset. Alle Nutzer profitieren automatisch.
- Mehrere Fragmente verschachteln: Link A → Link B → Link C. Der Scope-Stack verwaltet die Rückkehr.

## Troubleshooting

**Dialog endet nach dem Fragment, obwohl er weiterlaufen soll.**
`bReturnAfterExit = false` am Link-Node. Oder der Exit im Fragment hat `ExitStatus = Failed` und dein Host-Dialog reagiert darauf mit einem anderen Ausgang. Prüfe das Flag.

**Infinite-Loop zwischen Host und Fragment.**
Das Fragment linkt zurück auf den Host. Der Scope-Stack wird tiefer als Limit (16). Nutze Branches statt gegenseitige Links.

**Speaker im Fragment wird nicht aufgelöst.**
Das Fragment-Asset kennt den Speaker-Tag nicht. Füge den Speaker dort ebenfalls hinzu oder nutze den Token-Ansatz aus dem Pro-Tipp.
