# Wiederverwendbare Dialog-Fragmente

Wenn du merkst, dass in fünf Dialog-Assets exakt derselbe Abschluss-Satz steht („Pass auf dich auf, die Wälder sind gefährlich."), hast du ein Wiederverwendungs-Problem. Die Lösung: **Link-Node** oder **SubGraph-Node**. Dieses Rezept zeigt dir beide, mit einer klaren Empfehlung, wann welche.

## Szenario

Alle Händler im Spiel enden ihr Gespräch mit einem identischen Farewell-Block: *„Guten Weg, Reisender."* → optionale zufällige Warnung → Exit. Statt das in jedes Asset zu kopieren, lagern wir den Block aus.

## Variante A – Link-Node

### Aufbau

Ein separates Asset `DA_Common_Farewell` mit dem gemeinsamen Pattern. In jedem Händler-Dialog ein **Link-Node**, der auf dieses Asset zeigt.

### Graph-Mock-Ups

**`DA_Common_Farewell`**:

```
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

**`DA_Merchant_*`**:

```
... Händler-spezifisches Gespräch ...
   │
   ▼
[Link → DA_Common_Farewell]
   │
   ▼  (returnt nach Asset-Exit hierher)
[Exit: Completed]
```

### Schritt-für-Schritt

1. **Fragment-Asset anlegen**: `DA_Common_Farewell` mit obigem Inhalt. Speaker-Tag hier: `Dialogue.Speaker.CurrentSpeaker` (ein generischer Token – siehe Pro-Tipp unten).
2. **Im Händler-Dialog**: Vom letzten Händler-Node → *Create Node → Link*.
3. **Link-Node konfigurieren**:
   * `LinkedAsset`: `DA_Common_Farewell`
   * `EntryGuid`: *(leer = Asset-Entry)*
   * `bReturnAfterExit`: `true`
4. **Rückkehr verdrahten**: Der Link-Node hat einen Output, der feuert, **wenn das verlinkte Asset geexitet ist** (und der Scope-Stack zurückpoppt). Verbinde ihn mit deinem Abschluss-Exit.

### Wie der Scope-Stack das löst

Beim Auftreffen auf Link-Node push die Instance einen `FMayDialogueScopeEntry` auf ihren internen Stack:

```cpp
ScopeStack.Push({ Asset: "DA_Common_Farewell", ReturnNodeGuid: LinkNode.OutputNode });
```

Der Exit-Node im Fragment pop das oberste Frame und springt dorthin zurück. Details siehe [Instance & Lifecycle](../concepts/instance-lifecycle.md#links-scope-stack).

{% hint style="info" %}
**Pro-Tipp** für Speaker-Tags: Statt harter Tags im Fragment nutze generische Token wie `Dialogue.Speaker.CurrentSpeaker`, die im Host-Dialog-Asset gemappt werden. So kann das Farewell-Asset für jeden Händler funktionieren.
{% endhint %}

## Variante B – SubGraph-Node

SubGraph ist wie Link, nur dass das Fragment **nicht als eigenes Asset** existiert, sondern als Child-Graph innerhalb desselben Assets.

### Aufbau

Im Händler-Dialog ein SubGraph-Node, der intern einen eigenen kleinen Graph hält.

### Graph-Mock-Up

```
... Händler-Gespräch ...
   │
   ▼
┌────────────────────────────┐
│ SubGraph: "Farewell Block" │
│  ┌──────┐                   │
│  │Entry │                   │
│  └──┬───┘                   │
│     ▼                        │
│  [SayLine: "Guten Weg."]   │
│     │                        │
│  ┌──▼───┐                   │
│  │ Exit │                   │
│  └──────┘                   │
└────────────────────────────┘
   │
   ▼
[Exit: Completed]
```

### Wann SubGraph, wann Link?

| Kriterium | Link | SubGraph |
| --- | --- | --- |
| Wiederverwendung über Asset-Grenzen | Ja | Nein |
| Editor-Übersichtlichkeit in großen Assets | Nein | Ja (collapsible) |
| Eigenes Speakers-/Variables-Scope | Ja (neues Asset) | Nein (erbt vom Parent) |
| Referenz-Änderungen propagieren | Ja (zentral) | Nein (lokal) |
| Perforce-/Merge-Konflikte | Getrennt, weniger Konflikte | Alles in einer Datei |

**Faustregel**: Wiederverwendung → Link. Reine Graph-Ordnung innerhalb *einem* Asset → SubGraph.

## Kombination: Link + Parameter

Du kannst vor dem Link-Node [SetVariable](../nodes/actions/set-variable.md) benutzen, um dem Fragment Parameter mitzugeben (Dialogue-Scope teilt sich über den Link, weil die Variablen auf der Instance leben, nicht auf dem Asset):

```
... Händler-spezifisch ...
   │
   ▼
[SetVariable: "MerchantPriceMood" (Int) = 2]
   │
   ▼
[Link → DA_Common_Farewell]
```

Innerhalb des Fragments kannst du mit einem Branch auf diese Variable reagieren.

## Runtime-Verhalten

Für den Aufrufer ist das transparent. Es gibt nur **eine Instance** – das Fragment läuft nicht als neuer Dialog, sondern wird in dieselbe Instance „eingeklinkt". `OnDialogueStarted` feuert nicht erneut.

Auch wichtig:

* `GetCurrentNodeGUID()` liefert während des Fragments den Node im **Fragment-Asset** zurück, nicht im Parent.
* `GetActiveDialogueAsset()` liefert den Host. Das Fragment-Asset findest du nur über den Scope-Stack (siehe Subsystem-API).

## Troubleshooting

### Der Dialog endet nach dem Fragment, obwohl er weiterlaufen soll

* Exit-Node im Fragment setzt `ExitStatus = Completed`, aber `bReturnAfterExit` am Link-Node ist `false`. Dann wird der Host-Dialog mitbeendet. Setze das Flag oder leite das Fragment stattdessen über einen Return-Sonder-Exit.

### Infinite-Loop zwischen Host und Fragment

* Dein Fragment linkt zurück auf den Host (gegenseitig). Der Scope-Stack wird dann beliebig tief. Ab einer Tiefe von 16 bricht das Plugin ab. Das sollte ein Design-Fehler sein – nutze Branches statt gegenseitige Links.

### Speaker im Fragment wird nicht aufgelöst

* Der Host-Dialog kennt den Speaker-Tag, das Fragment nicht. Füge den Speaker auch im Fragment-Asset hinzu oder nutze den Token-Ansatz aus dem Pro-Tipp.

## Nächster Schritt

* [GAS-getriebener Dialog](gas-driven-dialogue.md) – Fragment-Inhalt abhängig von Spieler-Tags machen.
* [Eigene Nodes per Blueprint](../extension/custom-nodes.md) – wenn du einen wiederkehrenden Logik-Block brauchst, der mehr als nur ein Fragment ist.
