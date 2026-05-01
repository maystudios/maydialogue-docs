---
description: Eine PlayerChoice-Option nur anzeigen, wenn der Spieler einen bestimmten GameplayTag trägt.
---

# Choice nur sichtbar mit Tag

## Szenario

Ein Händler hat drei Kaufoptionen. Eine vierte Option – *„Ich bin Gilden-Mitglied, Rabatt bitte."* – soll nur erscheinen, wenn der Spieler den Tag `Guild.Member.Active` trägt. Nicht-Mitglieder sehen die Option überhaupt nicht; sie erraten nicht, dass es sie gibt. Dieses Muster ist der Klassiker für versteckte Dialog-Pfade.

## Was du lernst

- HasTag-Requirement direkt an einer Choice (nicht an einem Branch) hängen.
- `FailedAndHidden` vs. `FailedButVisible` unterscheiden.
- Mehrere Requirements an einer Choice kombinieren (AND-Logik).
- Choice-Tags für externe Systeme setzen.

## Voraussetzungen

- [Verzweigungen mit Bedingungen](branching-conditions.md) abgeschlossen.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: "Was darf ich dir anbieten?"]
   │
   ▼
[PlayerChoice]
   ├─ "Heiltrank kaufen."            (keine Req) → [SayLine: "10 Gold."] → [Exit]
   ├─ "Rüstung anschauen."           (keine Req) → [SayLine: "Hier entlang."] → [Exit]
   ├─ "Ich schaue nur."              (keine Req) → [Exit]
   └─ "Gilden-Rabatt bitte."         (Req: HasTag Guild.Member.Active, FailedAndHidden)
         └─► [SayLine: "Natürlich! 20 % für Gildenmitglieder."] → [Exit]
```

> 📸 **Bild-Platzhalter:** `choice-with-tag-requirement-graph-overview.png` — Übersichtsgraph des Händler-Dialogs.
> *Setup:* Asset `DA_Merchant_Shop` geöffnet. SayLine → PlayerChoice mit vier Choice-Sub-Nodes im Body. Letzte Choice hat Schloss-Icon neben dem Text. Im Preview-Runner (ohne Tag) sind nur drei Choices sichtbar.

## Schritt-für-Schritt

### 1. PlayerChoice anlegen

Asset: `DA_Merchant_Shop`. SayLine *„Was darf ich dir anbieten?"* → **PlayerChoice**-Node.

Im Details-Panel des PlayerChoice: vier Choices über **Add Choice** anlegen.

### 2. Erste drei Choices normal befüllen

Für jede der offenen Choices: `ChoiceText` eingeben, Output-Pin verdrahten.

### 3. Vierte Choice mit Requirement

Choice 4: `ChoiceText = "Gilden-Rabatt bitte."`. Unter **Requirements → Add → HasTag**:

| Property | Wert |
|----------|------|
| `RequiredTag` | `Guild.Member.Active` |
| `bCheckOnInstigator` | `true` |
| `FailureResult` | `FailedAndHidden` |

`FailedAndHidden` = die Choice taucht für Nicht-Mitglieder gar nicht in der Liste auf.

> 📸 **Bild-Platzhalter:** `choice-with-tag-requirement-choice-details.png` — Details-Panel der Choice 4 mit HasTag-Requirement.
> *Setup:* PlayerChoice-Node ausgewählt, Choice 4 im Details-Panel ausgeklappt. `ChoiceText = "Gilden-Rabatt bitte."`, darunter Requirements: `HasTag: RequiredTag = Guild.Member.Active, bCheckOnInstigator = true, FailureResult = FailedAndHidden`.

### 4. Output verdrahten

Choice 4-Output-Pin → SayLine *„Natürlich! 20 % für Gildenmitglieder."* → Exit.

### 5. Compile und testen

Preview-Runner: ohne Tag sieht man drei Choices. Tag im Debugger an Player-ASC setzen → vierte Choice erscheint.

## FailedAndHidden vs. FailedButVisible

| Einstellung | Verhalten | Wann nutzen |
|-------------|-----------|-------------|
| `FailedAndHidden` | Choice unsichtbar | Spieler soll nicht wissen, dass es die Option gibt |
| `FailedButVisible` | Choice sichtbar, aber ausgegraut + Tooltip | Spieler soll sehen, was er noch nicht freigespielt hat |

Die Tooltip-Meldung kommt aus `UnavailableReason` – leer lassen wenn keine Erklärung gewünscht.

## Mehrere Requirements kombinieren

Du kannst mehrere Requirements an einer Choice stapeln. Die Auswertung ist **AND**: alle müssen `Passed` sein.

Beispiel: Gilden-Mitglied UND genug Gold:

```text
Requirements:
  [0] HasTag(Guild.Member.Active)   FailedAndHidden
  [1] CheckAttribute(Gold >= 50)    FailedButVisible  UnavailableReason: "Zu wenig Gold."
```

Ergebnis: Nicht-Mitglieder sehen die Option nicht. Mitglieder ohne Gold sehen sie ausgegraut mit Hinweis.

## Blueprint-Triggering

Kein besonderer Code – normaler Dialogue-Start:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Merchant_Shop
   └─ ...
```

> 📸 **Bild-Platzhalter:** `choice-with-tag-requirement-ingame.png` — Preview-Runner mit drei sichtbaren Choices (Tag nicht gesetzt).
> *Setup:* Preview-Runner offen. Choice-List zeigt drei Einträge. Vierte Choice nicht sichtbar. Daneben: Tag am Player-ASC hinzufügen → Preview neu starten → vier Choices sichtbar.

## Variation / Weiter gehen

- Dasselbe Muster mit **Attribut statt Tag** → [Choice mit Attribut-Bedingung](choice-with-attribute-requirement.md).
- Choice-Tags setzen: `ChoiceTags` am Choice-Sub-Node befüllen. Externe Systeme (Quest-System) können auf „Spieler hat Choice mit Tag X gewählt" reagieren.
- `FailedButVisible` für ein klassisches RPG-Muster: Charisma-Option immer anzeigen, nur wählbar wenn Charisma ≥ 5.

## Troubleshooting

**Choice immer unsichtbar, obwohl Tag gesetzt.**
`bCheckOnInstigator = false` → Tag wird am NPC statt am Spieler geprüft. Setze auf `true`.

**Choice immer sichtbar, obwohl Tag fehlt.**
`FailureResult = FailedButVisible` statt `FailedAndHidden`. Oder `RequiredTag` hat Tippfehler – das Requirement greift ins Leere und gibt `Passed` zurück.

**Mehrere Requirements, Choice nie wählbar.**
AND-Logik: alle Requirements müssen `Passed` sein. Im Debugger jeden Requirement-Status einzeln prüfen.
