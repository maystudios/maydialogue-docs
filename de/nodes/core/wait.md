# Wait

Der Wait-Node pausiert den Dialog, bis eine oder mehrere Bedingungen eintreten: ein Timer läuft ab, ein GameplayEvent wird gefeuert, oder eine Bedingung wird wahr. Alle drei Modi können kombiniert werden.

## Wann setze ich ihn ein?

- Für dramatische Pausen zwischen zwei Sätzen.
- Wenn der Dialog auf eine Spieler-Aktion warten soll, bevor er weitergeht (z.B. "Geh zum Fenster").
- Für Cutscene-Timing: nach einer Animation oder einem Sound eine feste Pause einbauen.
- Als Polling-Schleife: warte, bis ein Attribut einen Schwellenwert erreicht.
- Mit AND-Semantik: erst wenn Zeit abgelaufen **und** Event gefeuert ist, geht es weiter.

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `WaitDuration` | `float` | `1.0` | Warte diese Anzahl Sekunden. `0` = Timer deaktiviert. |
| `WaitEventTag` | `FGameplayTag` | leer | Warte auf dieses GameplayEvent. Leer = Event-Wartemode deaktiviert. |
| `WaitCondition` | `UMayDialogueRequirement*` (Instanced) | leer | Pollt alle `ConditionCheckInterval` Sekunden. Leer = Bedingungsmode deaktiviert. |
| `ConditionCheckInterval` | `float` | `0.2` | Poll-Intervall in Sekunden. Nur aktiv wenn `WaitCondition` gesetzt. |
| `bRequireBoth` | `bool` | `false` | `false` = OR: jede aktivierte Bedingung reicht; `true` = AND: alle aktivierten Bedingungen müssen gleichzeitig erfüllt sein. |

{% hint style="info" %}
Eine Warte-Bedingung gilt als "aktiv", wenn sie konfiguriert ist: `WaitDuration > 0`, `WaitEventTag` valide, oder `WaitCondition` gesetzt. Wenn keine Bedingung aktiv ist, läuft der Node sofort durch.
{% endhint %}

> 📸 **Bild-Platzhalter:** `wait-node-graph.png` — Wait-Node im Graph mit Duration und EventTag.
> *Setup:* Wait-Node mit `WaitDuration = 2.0` und `WaitEventTag = Story.PlayerAtWindow`. Input-Pin verbunden mit `SayLine "Folge mir zum Fenster."`. Output-Pin verbunden mit `SayLine "Schau nach draußen..."`. Im Body des Wait-Nodes sichtbar: Duration- und Event-Pills.

> 📸 **Bild-Platzhalter:** `wait-details-panel.png` — Details-Panel des Wait-Nodes.
> *Setup:* Wait-Node auswählen. Im Details-Panel sichtbar: `WaitDuration = 2.0`, `WaitEventTag = Story.PlayerAtWindow`, `WaitCondition` (leer), `ConditionCheckInterval = 0.2`, `bRequireBoth = false`.

## Modi

| Modus | Konfiguration | Verhalten |
| --- | --- | --- |
| Nur Duration | `WaitDuration > 0` | Pausiert für die angegebene Zeit. |
| Nur Event | `WaitEventTag` valide | Wartet auf ein externes GameplayEvent. |
| Nur Condition | `WaitCondition` gesetzt | Pollt die Bedingung im Intervall. |
| OR (Standard) | mehrere aktiv, `bRequireBoth = false` | Geht weiter, wenn irgendeine Bedingung erfüllt ist. |
| AND | mehrere aktiv, `bRequireBoth = true` | Geht weiter, wenn alle aktiven Bedingungen erfüllt sind. |

## Mini-Beispiel

**Dramatische Pause:**

```text
[SayLine: NPC | "Ich... ich habe etwas zu beichten."]
  │
  ▼
[Wait: WaitDuration=1.5]
  │
  ▼
[SayLine: NPC | "Es war alles meine Schuld."]
```

**Warten auf externe Aktion:**

```text
[SayLine: NPC | "Folge mir zum Fenster."]
  │
  ▼
[Wait: WaitEventTag=Story.PlayerAtWindow]
  │
  ▼
[SayLine: NPC | "Siehst du das Lager da draußen?"]
```

> 📸 **Bild-Platzhalter:** `wait-example-graph.png` — Zwei Demo-Graphen nebeneinander: Timer-Modus und Event-Modus.
> *Setup:* Links: `SayLine "Ich habe etwas zu beichten."` → `Wait (Duration=1.5)` → `SayLine "Es war alles meine Schuld."`. Rechts: `SayLine "Folge mir."` → `Wait (EventTag=Story.PlayerAtWindow)` → `SayLine "Siehst du das Lager?"`. Beide Graphen komplett mit Verbindungen.

## Häufige Fallstricke

- **`WaitDuration = 0` mit leerem Event und leerem Condition**: Der Node läuft sofort durch — das ist korrekt, aber ein Wait ohne aktive Bedingung ist überflüssig.
- **Dialog während Wait abbrechen**: Timers und Event-Listener werden beim Abort/Cleanup automatisch aufgeräumt (über den Async-State-Mechanismus). Manuelles Aufräumen ist nicht nötig.
