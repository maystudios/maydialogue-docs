---
description: Einen GameplayEffect direkt aus dem Dialog anwenden – Buff, Debuff oder Heilung als Action-Node.
---

# GameplayEffect aus Dialog anwenden

## Szenario

Ein Priester heilt den Spieler im Gespräch. Eine Hexe verflucht ihn. Ein Schmied verleiht ihm temporär einen Damage-Buff. All das passiert über den **ApplyEffect**-Action-Node – ein einzelner Node im Graph, der einen beliebigen GameplayEffect auf Instigator oder Target anwendet. Kein Blueprint-Code nötig.

## Was du lernst

- ApplyEffect-Node anlegen und konfigurieren.
- Target (Instigator vs. Target-Actor) auswählen.
- Effect-Level und Context setzen.
- Mehrere Effekte hintereinander anwenden.

## Voraussetzungen

- GAS aktiv, AttributeSets registriert.
- GameplayEffect-Assets erstellt (z.B. `GE_PriestHeal`, `GE_WitchCurse`).

## Mini-Graph – Heilungs-Dialog

```text
[Entry]
   │
   ▼
[SayLine: Priester – "Möge das Licht dich heilen."]
   │
   ▼
[ApplyEffect: GE_PriestHeal → Instigator (Spieler)  Level: 1.0]
   │
   ▼
[SayLine: Priester – "Geh in Frieden."]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `apply-gameplay-effect-graph-overview.png` — Priester-Dialog mit ApplyEffect-Node.
> *Setup:* Asset `DA_Priest_Heal` geöffnet. Von links: Entry → SayLine (blau, Priester) → ApplyEffect-Node (lila Box, sichtbarer Effekt-Name) → SayLine → Exit. ApplyEffect-Node liegt prominent im Hauptfluss.

## Schritt-für-Schritt

### 1. GameplayEffect-Asset vorbereiten

In UE: neues Blueprint-Asset mit Parent `UGameplayEffect`. Für eine Heilung: `Instant` Duration, `Attribute: Health`, `Magnitude: 50`. Asset-Name: `GE_PriestHeal`.

### 2. Dialog-Asset anlegen

Asset: `DA_Priest_Heal`. Speaker: `Dialogue.Speaker.Priest`.

### 3. ApplyEffect-Node einfügen

Vom SayLine-Output → **Create Node → Apply Effect**.

| Property | Wert |
|----------|------|
| `EffectClass` | `GE_PriestHeal` |
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `EffectLevel` | `1.0` |
| `bApplyFromInstigator` | `true` |

`bApplyFromInstigator = true` → der Spieler ist Instigator des Effekts (Source = Spieler-ASC). Wichtig für Magnitude-Calculatoren, die von Source-Attributen lesen.

> 📸 **Bild-Platzhalter:** `apply-gameplay-effect-node-details.png` — Details-Panel des ApplyEffect-Nodes.
> *Setup:* ApplyEffect-Node ausgewählt. Details: `EffectClass = GE_PriestHeal`, `TargetParticipantTag = Dialogue.Participant.Player`, `EffectLevel = 1.0`, `bApplyFromInstigator = true (Checkbox)`.

### 4. Compile und testen

Im PIE: Dialog starten. Nach dem ApplyEffect-Node sollte das Health-Attribut des Spielers steigen. Im GAS-Debugger prüfen.

## Effekte auf den NPC anwenden

Wenn der Priester sich selbst bufft oder die Hexe den Spieler verflucht:

| Szenario | `TargetParticipantTag` | `bApplyFromInstigator` |
|----------|----------------------|-----------------------|
| Spieler heilt Spieler | `Dialogue.Participant.Player` | `true` |
| NPC bufft NPC | `Dialogue.Participant.Priest` | `false` |
| Hexe verflucht Spieler | `Dialogue.Participant.Player` | `false` (Hexe = Source) |

## Mehrere Effekte hintereinander

```text
[ApplyEffect: GE_WitchCurse_DotPoison → Player]
   │
   ▼
[ApplyEffect: GE_WitchCurse_DebuffSpeed → Player]
   │
   ▼
[SayLine: "Du wirst bereuen."]
```

Beide Nodes laufen nacheinander und sind im Graph als separate Schritte sichtbar – gut lesbar, leicht zu debuggen.

## ApplyEffect als SideEffect

Wenn der Effekt ein Nebenschritt ist (z.B. ein Buff der nebenher beim Betreten einer SayLine angewendet wird):

Am SayLine-Node **SideEffect → ApplyEffect** hinzufügen. Die SayLine läuft, der Effekt wird gleichzeitig angewendet – kein eigener Node im Hauptfluss.

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Priest_Heal
   └─ ...
```

> 📸 **Bild-Platzhalter:** `apply-gameplay-effect-ingame.png` — PIE mit Priester-Dialog, Health-Bar steigt sichtbar.
> *Setup:* PIE läuft. Dialog-Widget zeigt Priester-SayLine. GAS-Debug-Overlay (Taste ` → showdebug abilitysystem) zeigt Health steigende Animation nach ApplyEffect.

## Variation / Weiter gehen

- Effekt-Level dynamisch: `EffectLevel` aus einer Variable lesen (z.B. Priester-Level-Variable).
- Effekt nur wenn Health unter Schwelle → Requirement vor dem ApplyEffect-Node: [Choice mit Attribut-Bedingung](choice-with-attribute-requirement.md).
- Vollständiger GAS-Loop (Effekt + Tag + Variable) → [GAS-getriebener Dialog](gas-driven-dialogue.md).

## Troubleshooting

**Effekt zeigt keine Wirkung.**
`EffectClass` leer – häufigster Fehler. Target hat keinen ASC. Instant-GE aber `PostGameplayEffectExecute` klemmt das Attribut nicht.

**EffectLevel hat keine Wirkung.**
Der GameplayEffect nutzt keine Level-abhängige Magnitude (ScalableFloat). Level-Scaling muss explizit im GE-Asset konfiguriert sein.

**Effekt auf falschen Actor angewendet.**
`TargetParticipantTag` zeigt auf den NPC statt den Spieler. Tags im Speakers/Participants-Panel prüfen.
