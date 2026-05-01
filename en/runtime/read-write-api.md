---
description: Read and write dialogue state from outside — variables, choices, ForceAdvance.
---

# Read/Write API

External systems can interact with a running dialogue: read and write variables, programmatically select choices, skip the advance timer. Useful for cheat menus, tutorial scripts, auto-play tests, and analytics.

All methods are available directly on the subsystem as Blueprint-callable. You do not need to hold an instance reference.

---

## Reading

| Function (Blueprint name) | Description |
|---|---|
| `Get Active Dialogue Asset` | The asset of the running instance. `nullptr` if nothing is running. |
| `Get Current Node GUID` | GUID of the currently executing Node. Useful for telemetry. |
| `Get Active Participants` | Array of all participating actors. |
| `Get Dialogue Variable (As String)` | Reads a dialogue-scope variable by name and type. |
| `Get Participant Variable (As String)` | Reads a participant-scope variable (persistent memory). |
| `Get Pending Choices` | Current Choice array (empty if no PlayerChoice is active). |

### Blueprint: Reading a variable

> 📸 **Image placeholder:** `read-variable-bp.png` — BP graph: Get Dialogue Variable on the subsystem.
> *Setup:* Debug widget Blueprint. `Get MayDialogue Subsystem` → `Get Dialogue Variable (As String)` (DisplayName from the header). Pins: `Name` = "AngerLevel" (String literal), `Type` = Int. Out parameter `Out Value As String` goes into `String To Int` → `Set Text` on a Text widget. Below: `Return Value` (bool) → Branch for "variable exists".

```text
[Get MayDialogue Subsystem]
    │
    ▼
[Get Dialogue Variable (As String)]
  ├─ Name: "AngerLevel"
  ├─ Type: Int
  └─ Out Value As String → (process string further)
       │ Return Value (bool: found?)
       ▼
  [Branch]
    ├─ True  → Display value
    └─ False → Use default value
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

## Writing

| Function (Blueprint name) | Description |
|---|---|
| `Set Dialogue Variable (From String)` | Sets a dialogue-scope variable. Immediately triggers `OnVariableChanged`. |
| `Set Participant Variable (From String)` | Sets a participant-scope variable (persistent memory). |
| `Select Choice` | Programmatically selects a Choice by index. |
| `Force Advance` | Skips the current advance wait (timer, voice end). |

### Blueprint: Setting a variable

> 📸 **Image placeholder:** `set-variable-bp.png` — BP graph: Cheat menu sets a dialogue variable.
> *Setup:* Cheat panel Widget Blueprint. `Button On Clicked` → `Get MayDialogue Subsystem` → `Set Dialogue Variable (From String)`. Pins: `Name` = "CheatMode" (literal), `Type` = Bool, `Value As String` = "true". `Return Value` (bool: success?) → Print String.

```text
[Button On Clicked: Cheat All Choices]
    │
    ▼
[Get MayDialogue Subsystem] → [Set Dialogue Variable (From String)]
  ├─ Name:           "CheatMode"
  ├─ Type:           Bool
  └─ Value As String: "true"
```

### C++: Auto-player for tests

```cpp
// Automatically click through all dialogues (e.g. in automation tests)
void UAutoPlayer::Tick(float DeltaSeconds)
{
    auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();
    if (!Sub || !Sub->IsAnyDialogueActive()) return;

    // If choices are available: select the first one
    TArray<FMayDialogueChoiceEntry> Choices = Sub->GetPendingChoices();
    if (Choices.Num() > 0)
    {
        Sub->SelectChoice(0);
        return;
    }

    // Otherwise: skip the advance
    Sub->ForceAdvance();
}
```

---

## SelectChoice and ForceAdvance

| Function | When | Notes |
|---|---|---|
| `Select Choice` | When `Status == WaitingForChoice` | Runs the normal Choice cascade (SideEffects, Requirements). No shortcut. |
| `Force Advance` | When `Status == WaitingForAdvance` | Skips timer and voice end, goes to the next Node immediately. |

{% hint style="warning" %}
`Force Advance` ignores the configured advance mode. Only use it for tests, cheats, or tutorial scripting — not as normal player input.
{% endhint %}

---

## String Serialization of Variable Types

The variable API works with string representations:

| Type | String format | Example |
|---|---|---|
| `Bool` | `"true"` / `"false"` | `"true"` |
| `Int` | Integer as text | `"42"` |
| `Float` | Decimal as text | `"3.14"` |
| `String` | Any text | `"Hello"` |
| `Tag` | Full tag path | `"Dialogue.Mood.Friendly"` |

The subsystem handles the parsing — you just pass the type enum along.

---

## Use Cases

| Scenario | Solution |
|---|---|
| Cheat menu unlocks all Choices | `Set Dialogue Variable` → set Requirement variable to passed value |
| Tutorial script clicks through automatically | `Force Advance` + `Select Choice(0)` in Tick |
| Analytics logs dialogue state | `Get Active Dialogue Asset`, `Get Current Node GUID`, `Get Dialogue Variable` |
| Debug overlay shows all variables | `Get Dialogue Variable` in a loop over known variable names |
| Player variable should control conversation | `Set Participant Variable` → read by Requirements of the next Choices |

---

## UMayDialogueVariablesLibrary — Container Access

If you want to access an `FMayDialogueVariables` container directly (e.g. from a saved instance or a custom Node state) without using the string-based subsystem path, `UMayDialogueVariablesLibrary` is available:

| Function | Type | Description |
|---|---|---|
| `Get Dialogue Variable` | Callable | Read a variable by name from a container |
| `Set Dialogue Variable` | Callable | Write a variable to a container |
| `Copy Dialogue Variables` | Callable | Copy variables between two containers |

```text
[Get Dialogue Variable]   (Category: MayDialogue|Variables)
  ├─ Container: (FMayDialogueVariables reference)
  ├─ Name:      "AngerLevel"
  ├─ Type:      Int
  └─ Out Value As String → (process further)
```

> **Note:** `Copy Dialogue Variables` currently returns `false` (stub implementation). Use `Get`+`Set` in a loop instead if you need to port variables.

---

## Directly on the Instance (Typed)

If you already have an instance reference, you can use typed getters/setters directly — without string conversion:

```cpp
Inst->SetDialogueVariableInt("AngerLevel", 5);
Inst->SetDialogueVariableBool("HasMetBefore", true);

int32 Anger;
Inst->GetDialogueVariableInt("AngerLevel", Anger);
```

> 📸 **Image placeholder:** `instance-variable-bp.png` — BP graph: Set typed variable directly on the instance.
> *Setup:* Blueprint that has stored an instance reference. `Set Dialogue Variable Bool` (directly on the Instance object): `Var Name` = "HasMetBefore", `Value` = True. Below: `Set Dialogue Variable Int`: `Var Name` = "MeetingCount", `Value` = 1.
