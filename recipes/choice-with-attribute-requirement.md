---
description: Eine PlayerChoice-Option nur wählbar machen, wenn ein GAS-Attribut einen Schwellenwert erreicht.
---

# Choice mit Attribut-Bedingung

## Szenario

Ein Alchemist bietet dem Spieler einen starken Trank an. Die Kaufoption soll immer sichtbar sein – aber nur wählbar, wenn der Spieler mindestens 50 Stamina hat. Ist Stamina unter 50, sieht der Spieler die Choice ausgegraut mit dem Hinweis *„Du bist zu erschöpft."* Das ist das klassische RPG-Muster: Spieler weiß, was er noch nicht erreicht hat.

## Was du lernst

- CheckAttribute-Requirement an einer Choice konfigurieren.
- `FailedButVisible` mit `UnavailableReason` kombinieren.
- Unterschied zu `FailedAndHidden` (Choice-Tag-Requirement-Rezept).
- Attribut-Vergleichsoperatoren im Überblick.

## Voraussetzungen

- [Verzweigungen mit Bedingungen](branching-conditions.md) abgeschlossen.
- GAS aktiv, AttributeSet mit `Stamina` registriert und am Spieler-ASC.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: "Was suchst du, Wanderer?"]
   │
   ▼
[PlayerChoice]
   ├─ "Den gewöhnlichen Trank."   (keine Req) → [Exit: Completed]
   └─ "Den starken Trank."        (Req: Stamina >= 50, FailedButVisible)
         └─► [SayLine: "Gut. 30 Gold."] → [Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `choice-with-attribute-requirement-graph-overview.png` — Graph mit PlayerChoice und Attribut-gesperrter Choice.
> *Setup:* Asset `DA_Alchemist_Shop` geöffnet. SayLine → PlayerChoice mit zwei Choices. Zweite Choice hat Schloss-Icon (gelb = FailedButVisible). Im Details-Panel: CheckAttribute-Requirement sichtbar.

## Schritt-für-Schritt

### 1. Asset und Speaker anlegen

Asset: `DA_Alchemist_Shop`. Speaker: `Dialogue.Speaker.Alchemist`.

### 2. PlayerChoice mit zwei Choices

SayLine *„Was suchst du, Wanderer?"* → PlayerChoice. Zwei Choices über **Add Choice**.

Choice 1: `ChoiceText = "Den gewöhnlichen Trank."` – kein Requirement.

Choice 2: `ChoiceText = "Den starken Trank."` – mit Requirement (nächster Schritt).

### 3. CheckAttribute-Requirement konfigurieren

An Choice 2 unter **Requirements → Add → CheckAttribute**:

| Property | Wert |
|----------|------|
| `Attribute` | `UVHSAttributeSet::Stamina` (Dropdown) |
| `ComparisonOp` | `>=` |
| `ComparisonValue` | `50.0` |
| `FailureResult` | `FailedButVisible` |
| `UnavailableReason` | *„Du bist zu erschöpft."* |
| `bCheckOnInstigator` | `true` |

> 📸 **Bild-Platzhalter:** `choice-with-attribute-requirement-details.png` — Details-Panel der zweiten Choice mit CheckAttribute-Requirement.
> *Setup:* PlayerChoice ausgewählt, Choice 2 ausgeklappt. `ChoiceText = "Den starken Trank."`. Requirements: `CheckAttribute: Attribute = Stamina, ComparisonOp = >=, ComparisonValue = 50.0, FailureResult = FailedButVisible, UnavailableReason = "Du bist zu erschöpft."`.

### 4. Outputs verdrahten

Choice 1 → Exit. Choice 2 → SayLine *„Gut. 30 Gold."* → Exit.

### 5. Compile und testen

Im Preview-Runner: Stamina unter 50 → Choice 2 sichtbar aber ausgegraut, Hover zeigt den Reason-Text. Stamina über 50 → Choice wählbar.

## Verfügbare Vergleichsoperatoren

| Operator | Bedeutung |
|----------|-----------|
| `==` | Exakt gleich |
| `!=` | Ungleich |
| `<` | Kleiner als |
| `<=` | Kleiner gleich |
| `>` | Größer als |
| `>=` | Größer gleich |

## Blueprint-Triggering

Normaler Dialogue-Start:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Alchemist_Shop
   └─ ...
```

> 📸 **Bild-Platzhalter:** `choice-with-attribute-requirement-ingame.png` — Preview-Runner mit ausgegrautem Choice.
> *Setup:* Preview-Runner mit `Stamina = 30`. Choice-List zeigt beide Choices, zweite ausgegraut. Hover-Tooltip: `"Du bist zu erschöpft."`. Stamina im Debugger auf 60 setzen → Choice aktiv.

{% hint style="info" %}
**C++-Variante:** Das Requirement kannst du auch direkt in C++ bauen. Erstelle eine Subklasse von `UMayDialogueRequirement` und überschreibe `CheckRequirement()`. Das Attribute kannst du dort über `AbilitySystemComponent->GetNumericAttribute(Attribute)` lesen.
{% endhint %}

## Variation / Weiter gehen

- Dasselbe Muster für **Gold-Kosten**: `CheckAttribute(Currency.Gold >= 30)`, `FailedButVisible`, Reason *„Zu wenig Gold."*
- Mehrere Attribute kombinieren (AND): zweites Requirement an dieselbe Choice hängen.
- Bei Erfolg Stamina abziehen: SideEffect an der Choice → `ApplyEffect(GE_StaminaCost)`.

## Troubleshooting

**Choice immer ausgegraut, obwohl Attribut hoch genug.**
`bCheckOnInstigator = false` → Attribut wird am NPC-ASC geprüft, nicht am Spieler. Setze auf `true`. Außerdem: Attribut-Dropdown leer oder falsch – das AttributeSet muss am ASC registriert sein.

**Attribut-Dropdown zeigt das Attribut nicht.**
Das AttributeSet ist am ASC des Spielers nicht registriert. `UAbilitySystemComponent::AddAttributeSetSubobject()` oder `DefaultStartingData` in GameplayAbility-Settings prüfen.

**UnavailableReason wird nicht im Widget angezeigt.**
Das Dialog-Widget muss `UnavailableReason` aus den Choice-Daten lesen und anzeigen. Im Standard-Slate-Debug-Widget ist das implementiert; bei eigenem UMG-Widget muss du den Tooltip selbst binden.
