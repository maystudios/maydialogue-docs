---
description: Dialog-State von außen lesen und schreiben — Variablen, Choices, ForceAdvance.
---

# Read/Write-API

Externe Systeme können in einen laufenden Dialog eingreifen: Variablen lesen und setzen, Choices programmatisch auswählen, den Advance-Timer überspringen. Nützlich für Cheat-Menüs, Tutorial-Skripte, Auto-Play-Tests und Analytics.

Alle Methoden sind direkt am Subsystem als Blueprint-Callable verfügbar. Du brauchst keine Instance-Referenz zu halten.

---

## Lesen

| Funktion (Blueprint-Name) | Beschreibung |
|---|---|
| `Get Active Dialogue Asset` | Das Asset der laufenden Instance. `nullptr` wenn nichts läuft. |
| `Get Current Node GUID` | GUID des aktuell ausgeführten Nodes. Nützlich für Telemetrie. |
| `Get Active Participants` | Array aller teilnehmenden Actors. |
| `Get Dialogue Variable (As String)` | Liest eine Dialog-Scope-Variable nach Name und Typ. |
| `Get Participant Variable (As String)` | Liest eine Participant-Scope-Variable (persistentes Memory). |
| `Get Pending Choices` | Aktueller Choice-Array (leer wenn kein PlayerChoice aktiv). |

### Blueprint: Variable lesen

> 📸 **Bild-Platzhalter:** `read-variable-bp.png` — BP-Graph: Get Dialogue Variable am Subsystem.
> *Setup:* Debug-Widget-Blueprint. `Get MayDialogue Subsystem` → `Get Dialogue Variable (As String)` (DisplayName aus dem Header). Pins: `Name` = "AngerLevel" (String-Literal), `Type` = Int. Out-Parameter `Out Value As String` geht in `String To Int` → `Set Text` auf einem Text-Widget. Darunter `Return Value` (bool) → Branch für "Variable existiert".

```text
[Get MayDialogue Subsystem]
    │
    ▼
[Get Dialogue Variable (As String)]
  ├─ Name: "AngerLevel"
  ├─ Type: Int
  └─ Out Value As String → (String weiterverarbeiten)
       │ Return Value (bool: gefunden?)
       ▼
  [Branch]
    ├─ True  → Wert anzeigen
    └─ False → Default-Wert nutzen
```

### C++

```cpp
UMayDialogueSubsystem* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();
if (!Sub->IsAnyDialogueActive()) return;

FString AngerStr;
if (Sub->GetDialogueVariable("AngerLevel", EMayDialogueVariableType::Int, AngerStr))
{
    int32 Anger = FCString::Atoi(*AngerStr);
    UE_LOG(LogMyGame, Log, TEXT("Anger: %d"), Anger);
}
```

---

## Schreiben

| Funktion (Blueprint-Name) | Beschreibung |
|---|---|
| `Set Dialogue Variable (From String)` | Setzt eine Dialog-Scope-Variable. Triggert sofort `OnVariableChanged`. |
| `Set Participant Variable (From String)` | Setzt eine Participant-Scope-Variable (persistentes Memory). |
| `Select Choice` | Wählt eine Choice per Index programmatisch aus. |
| `Force Advance` | Überspringt den aktuellen Advance-Wait (Timer, Voice-Ende). |

### Blueprint: Variable setzen

> 📸 **Bild-Platzhalter:** `set-variable-bp.png` — BP-Graph: Cheat-Menü setzt eine Dialog-Variable.
> *Setup:* Cheat-Panel-Widget-Blueprint. `Button On Clicked` → `Get MayDialogue Subsystem` → `Set Dialogue Variable (From String)`. Pins: `Name` = "CheatMode" (Literal), `Type` = Bool, `Value As String` = "true". `Return Value` (bool: erfolgreich?) → Print String.

```text
[Button On Clicked: Cheat All Choices]
    │
    ▼
[Get MayDialogue Subsystem] → [Set Dialogue Variable (From String)]
  ├─ Name:           "CheatMode"
  ├─ Type:           Bool
  └─ Value As String: "true"
```

### C++: Auto-Player für Tests

```cpp
// Automatisch alle Dialoge durchklicken (z.B. in Automation-Tests)
void UAutoPlayer::Tick(float DeltaSeconds)
{
    auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();
    if (!Sub || !Sub->IsAnyDialogueActive()) return;

    // Wenn Choices vorhanden: erste auswählen
    TArray<FMayDialogueChoiceEntry> Choices = Sub->GetPendingChoices();
    if (Choices.Num() > 0)
    {
        Sub->SelectChoice(0);
        return;
    }

    // Sonst: Advance überspringen
    Sub->ForceAdvance();
}
```

---

## SelectChoice und ForceAdvance

| Funktion | Wann | Besonderheiten |
|---|---|---|
| `Select Choice` | Wenn `Status == WaitingForChoice` | Führt die normale Choice-Kaskade aus (SideEffects, Requirements). Kein Shortcut. |
| `Force Advance` | Wenn `Status == WaitingForAdvance` | Überspringt Timer und Voice-Ende, geht sofort zum nächsten Node. |

{% hint style="warning" %}
`Force Advance` ignoriert den konfigurierten Advance-Mode. Nutze ihn nur für Tests, Cheats oder Tutorial-Scripting — nicht als normalen Spieler-Input.
{% endhint %}

---

## String-Serialisierung der Variablen-Typen

Die Variable-API arbeitet mit String-Repräsentationen:

| Typ | String-Format | Beispiel |
|---|---|---|
| `Bool` | `"true"` / `"false"` | `"true"` |
| `Int` | Ganzzahl als Text | `"42"` |
| `Float` | Dezimalzahl als Text | `"3.14"` |
| `String` | Beliebiger Text | `"Hallo"` |
| `Tag` | Voller Tag-Pfad | `"Dialogue.Mood.Friendly"` |

Das Parsing übernimmt das Subsystem — du gibst nur den Typ-Enum mit an.

---

## Einsatzfälle

| Szenario | Lösung |
|---|---|
| Cheat-Menü entsperrt alle Choices | `Set Dialogue Variable` → Requirement-Variable auf Passed-Wert setzen |
| Tutorial-Skript klickt automatisch durch | `Force Advance` + `Select Choice(0)` im Tick |
| Analytics loggt Dialog-Zustand | `Get Active Dialogue Asset`, `Get Current Node GUID`, `Get Dialogue Variable` |
| Debug-Overlay zeigt alle Variablen | `Get Dialogue Variable` in Schleife über bekannte Variablennamen |
| Spieler-Variable soll Gespräch steuern | `Set Participant Variable` → wird von Requirements der nächsten Choices gelesen |

---

## UMayDialogueVariablesLibrary — Container-Zugriff

Wenn du direkt auf einen `FMayDialogueVariables`-Container (z.B. aus einer gespeicherten Instance oder einem Custom-Node-State) zugreifen willst, ohne den String-basierten Subsystem-Pfad zu nutzen, steht `UMayDialogueVariablesLibrary` bereit:

| Funktion | Art | Beschreibung |
|---|---|---|
| `Get Dialogue Variable` | Callable | Variable nach Name aus einem Container lesen |
| `Set Dialogue Variable` | Callable | Variable in einen Container schreiben |
| `Copy Dialogue Variables` | Callable | Variablen zwischen zwei Containern kopieren |

```text
[Get Dialogue Variable]   (Kategorie: MayDialogue|Variables)
  ├─ Container: (FMayDialogueVariables-Referenz)
  ├─ Name:      "AngerLevel"
  ├─ Type:      Int
  └─ Out Value As String → (weiterverarbeiten)
```

> **Hinweis:** `Copy Dialogue Variables` gibt aktuell `false` zurück (Stub-Implementierung). Nutze stattdessen `Get`+`Set` in einer Schleife, wenn du Variablen portieren willst.

---

## Auf der Instance direkt (typisiert)

Wenn du bereits eine Instance-Referenz hast, kannst du typisierte Getter/Setter direkt nutzen — ohne String-Konvertierung:

```cpp
Inst->SetDialogueVariableInt("AngerLevel", 5);
Inst->SetDialogueVariableBool("HasMetBefore", true);

int32 Anger;
Inst->GetDialogueVariableInt("AngerLevel", Anger);
```

> 📸 **Bild-Platzhalter:** `instance-variable-bp.png` — BP-Graph: Typisierte Variable direkt an der Instance setzen.
> *Setup:* Blueprint das eine Instance-Referenz gespeichert hat. `Set Dialogue Variable Bool` (direkt am Instance-Object): `Var Name` = "HasMetBefore", `Value` = True. Darunter `Set Dialogue Variable Int`: `Var Name` = "MeetingCount", `Value` = 1.
