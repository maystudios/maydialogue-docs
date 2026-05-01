# SideEffect

A SideEffect is an inline action that executes as soon as the parent Node is entered — or, in the case of a Choice, when the option is selected. SideEffects live as pills in the Node body and keep the graph clean without needing their own Node box.

## When should I use it?

- When a variable should be set or a tag assigned alongside entering a SayLine.
- When a gameplay action should be triggered on selecting a Choice (e.g. applying a GAS effect).
- For two or three simultaneous inline actions at the same point — for more, a dedicated Action Node is worthwhile.
- On the Entry Node to initialize dialogue state at start.
- On the Exit Node to execute a final action before the dialogue ends.

## Properties (Base Class)

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `Description` | `FText` | empty | Editor label of the pill in the graph. Also displayed in the Details panel. |

## Built-in SideEffects (GAS Integration)

These SideEffects are available out of the box — no additional setup needed. They access the Gameplay Ability System if your project uses it.

| Class | Action | Key Properties |
| --- | --- | --- |
| `UMayDlgSideEffect_AddTag` | Add a GameplayTag | `TagToAdd`, `bAddToInstigator` |
| `UMayDlgSideEffect_RemoveTag` | Remove a GameplayTag | `TagToRemove`, `bRemoveFromInstigator` |
| `UMayDlgSideEffect_ApplyEffect` | Apply a GAS GameplayEffect | `EffectClass`, `EffectLevel`, `bApplyToInstigator` |
| `UMayDlgSideEffect_TriggerCue` | Fire a GameplayCue | `CueTag`, `bTriggerOnInstigator` |

Details: [GAS → Actions](../../gas/actions.md).

> 📸 **Image placeholder:** `sideeffect-pills-on-sayline.png` — SayLine Node with three SideEffect pills in the body.
> *Setup:* Select SayLine Node `"Here is your reward, hero."`. Visible in the Node body: three SideEffect pills: `ApplyEffect: GE_ReputationUp Lv.2`, `AddTag: Story.Guard.Grateful`, `TriggerCue: GameplayCue.UI.RewardFlash`. Input pin connected on the left, Output pin connected on the right.

> 📸 **Image placeholder:** `sideeffect-details-panel.png` — Details panel of an ApplyEffect SideEffect.
> *Setup:* Expand the SideEffect entry (type `UMayDlgSideEffect_ApplyEffect`). Visible: `Description = "Increase reputation"`, `EffectClass = GE_ReputationUp`, `EffectLevel = 2.0`, `bApplyToInstigator = true`.

> 📸 **Image placeholder:** `sideeffect-on-choice-pill.png` — Choice pill with SideEffect pill on a PlayerChoice Node.
> *Setup:* PlayerChoice Node. Choice pill `"Attack!"` expanded. Inside: SideEffect pill `ApplyEffect: GE_Adrenaline`. Shows nesting: PlayerChoice → Choice → SideEffect.

## Mini Example

**SayLine with multiple inline actions:**

```text
[SayLine: NPC | "Here is your reward, hero."]
  SideEffects:
    + ApplyEffect: GE_ReputationUp, Level=2, bApplyToInstigator=true
    + AddTag: Story.Guard.Grateful
    + TriggerCue: GameplayCue.UI.RewardFlash
```

All three actions run when the SayLine is entered, **before** the text is displayed.

**Choice with SideEffect:**

```text
[PlayerChoice]
  Choice 0 "I'll fight!"
    SideEffect: ApplyEffect GE_Adrenaline
    ──► [SayLine: "Then let's go!"]
```

The effect is applied when the Choice is clicked, before the dialogue jumps to the SayLine.

> 📸 **Image placeholder:** `sideeffect-example-graph.png` — SayLine with SideEffects and transition to Exit.
> *Setup:* `Entry` → `SayLine "Here is your reward."` (three SideEffect pills in body) → `Exit: Completed`. Connections visible. In the SayLine's Details panel, the three SideEffects expanded.

## Action Node vs. SideEffect

| Criterion | Action Node | SideEffect |
| --- | --- | --- |
| Main action of this step? | Yes | No |
| Should be visible in the graph as its own box? | Yes | No |
| Set a debugger breakpoint on it? | Yes | No (only on parent Node) |
| Multiple actions at the same moment? | Gets cluttered | Ideal |

**Rule of thumb**: Up to two inline actions → SideEffect pills. Three or more and/or the action is the actual purpose of the step → dedicated Action Node.

## Common Pitfalls

- **SideEffects run before `ExecuteNode`**: If you want to react in a SideEffect to state that is only changed by the Node itself, the order will be wrong. SideEffects always run first.
- **`bAddToInstigator` / `bApplyToInstigator` forgotten**: Without this flag, the action affects the dialogue's Target (e.g. NPC), not the player. Set it to `true` for player-related effects.

## Extending

{% hint style="success" %}
**Custom SideEffect logic in Blueprint:**

1. Right-click in Content Browser → Blueprint Class → Parent: `UMayDialogueSideEffect`.
2. Override Event `ExecuteSideEffect` — put your gameplay logic here.
3. Optional: override `ExecuteClientSideEffect` for purely cosmetic actions (sounds, particles).
4. Compile Blueprint.
5. In the Details panel of a Node/Choice → "Add SideEffect" → your new class appears in the list.

Details: [Extension → Custom SideEffects](../../extension/custom-side-effects.md).
{% endhint %}

```cpp
// C++ override (Advanced):
virtual void ExecuteSideEffect_Implementation(const FMayDialogueContext& Context) override;
virtual void ExecuteClientSideEffect_Implementation(const FMayDialogueContext& Context) override;
```
