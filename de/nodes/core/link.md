# Link

Der Link-Node springt in ein anderes Dialog-Asset. Optional kehrt die Ausführung nach dessen Ende in den aktuellen Dialog zurück — so lassen sich wiederverwendbare Dialog-Fragmente auslagern.

## Wann setze ich ihn ein?

- Für Dialog-Bausteine, die mehrere Assets nutzen (z.B. eine Standard-Verabschiedung, ein häufiges NPC-Fragment).
- Als "Kapitel-Wechsel": der aktuelle Dialog endet, ein neuer startet ohne Return (`bReturnAfterTarget = false`).
- Um lange Dialoge in kleinere Assets aufzuteilen, die separat bearbeitet werden können.
- Für verschachtelte Dialog-Sequenzen, die nach Abschluss wieder zum Aufrufer zurückkehren.

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `TargetDialogueAsset` | `TSoftObjectPtr<UMayDialogueAsset>` | leer | Das Ziel-Asset. Soft-Referenz — wird erst bei Bedarf geladen. |
| `bReturnAfterTarget` | `bool` | `true` | `true` = nach Ende des Ziel-Assets zurück zum nächsten Node; `false` = Ziel ersetzt den aktuellen Dialog komplett (kein Return). |

{% hint style="info" %}
Das Ziel-Asset wird immer an dessen einzigem Entry-Node gestartet. Du kannst keinen anderen Einstiegspunkt wählen.
{% endhint %}

> 📸 **Bild-Platzhalter:** `link-node-graph.png` — Link-Node mit Ziel-Asset und Return-Pin.
> *Setup:* Link-Node mit `TargetDialogueAsset = DA_CommonFarewell` und `bReturnAfterTarget = true`. Input-Pin verbunden mit `SayLine "Komm morgen wieder."` links. Output-Pin (Return) verbunden mit `Exit Completed` rechts. Im Body des Link-Nodes: Asset-Name sichtbar.

> 📸 **Bild-Platzhalter:** `link-details-panel.png` — Details-Panel des Link-Nodes.
> *Setup:* Link-Node auswählen. Im Details-Panel sichtbar: `TargetDialogueAsset = DA_CommonFarewell`, `bReturnAfterTarget = true`.

## Mini-Beispiel

**Wiederverwendbares Fragment:**

```text
DA_GuardGreeting:
  [Entry] → [SayLine: "Willkommen."] → [Link → DA_CommonFarewell, bReturnAfterTarget=true]
                                          │ (Return)
                                          ▼
                                        [Exit: Completed]

DA_CommonFarewell:
  [Entry] → [SayLine: "Gute Reise."] → [Exit]
```

**Kapitel-Wechsel ohne Return:**

```text
[SayLine: "Das Abenteuer beginnt jetzt."] → [Link → DA_Chapter2_Opening, bReturnAfterTarget=false]
```

> 📸 **Bild-Platzhalter:** `link-example-graph.png` — Zwei Assets: Aufrufer mit Link-Node und Ziel-Asset.
> *Setup:* Links Asset `DA_GuardGreeting`: `Entry` → `SayLine "Willkommen."` → `Link (DA_CommonFarewell, Return=true)` → `Exit Completed`. Rechts Asset `DA_CommonFarewell`: `Entry` → `SayLine "Gute Reise."` → `Exit`. Annotation mit Pfeil "Return nach Exit".

## Häufige Fallstricke

- **Zirkuläre Links** (Asset A → Asset B → Asset A): Theoretisch möglich und führen zu einer Endlosschleife. Begrenze sie mit einem Requirement oder einer Variable.
- **`TargetDialogueAsset` leer**: Der Compiler meldet einen Error. Der Link-Node braucht immer ein gültiges Ziel.

## Erweitern

{% hint style="info" %}
Link vs. SubGraph: Wenn das Fragment nur innerhalb dieses Assets gebraucht wird, nutze stattdessen [SubGraph](sub-graph.md) — kein separates Asset nötig, einfacher im Editor zu navigieren.
{% endhint %}
