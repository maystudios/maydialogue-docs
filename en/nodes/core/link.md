# Link

The Link Node jumps to another dialogue asset. Optionally, execution returns to the current dialogue after it ends — allowing reusable dialogue fragments to be stored separately.

## When should I use it?

- For dialogue building blocks used by multiple assets (e.g. a standard farewell, a common NPC fragment).
- As a "chapter transition": the current dialogue ends, a new one starts without return (`bReturnAfterTarget = false`).
- To split long dialogues into smaller assets that can be edited separately.
- For nested dialogue sequences that return to the caller after completion.

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `TargetDialogueAsset` | `TSoftObjectPtr<UMayDialogueAsset>` | empty | The target asset. Soft reference — loaded on demand. |
| `bReturnAfterTarget` | `bool` | `true` | `true` = return to the next Node after the target asset ends; `false` = target replaces the current dialogue completely (no return). |

{% hint style="info" %}
The target asset always starts at its single Entry Node. You cannot choose a different entry point.
{% endhint %}

> 📸 **Image placeholder:** `link-node-graph.png` — Link Node with target asset and return pin.
> *Setup:* Link Node with `TargetDialogueAsset = DA_CommonFarewell` and `bReturnAfterTarget = true`. Input pin connected to `SayLine "Come back tomorrow."` on the left. Output pin (Return) connected to `Exit Completed` on the right. In the Link Node body: asset name visible.

> 📸 **Image placeholder:** `link-details-panel.png` — Details panel of the Link Node.
> *Setup:* Select the Link Node. In the Details panel: `TargetDialogueAsset = DA_CommonFarewell`, `bReturnAfterTarget = true`.

## Mini Example

**Reusable fragment:**

```text
DA_GuardGreeting:
  [Entry] → [SayLine: "Welcome."] → [Link → DA_CommonFarewell, bReturnAfterTarget=true]
                                        │ (Return)
                                        ▼
                                      [Exit: Completed]

DA_CommonFarewell:
  [Entry] → [SayLine: "Safe travels."] → [Exit]
```

**Chapter transition without return:**

```text
[SayLine: "The adventure begins now."] → [Link → DA_Chapter2_Opening, bReturnAfterTarget=false]
```

> 📸 **Image placeholder:** `link-example-graph.png` — Two assets: caller with Link Node and target asset.
> *Setup:* Left asset `DA_GuardGreeting`: `Entry` → `SayLine "Welcome."` → `Link (DA_CommonFarewell, Return=true)` → `Exit Completed`. Right asset `DA_CommonFarewell`: `Entry` → `SayLine "Safe travels."` → `Exit`. Annotation with arrow "Return after Exit".

## Common Pitfalls

- **Circular links** (Asset A → Asset B → Asset A): Theoretically possible and lead to an infinite loop. Limit them with a Requirement or a variable.
- **`TargetDialogueAsset` empty**: The compiler reports an error. The Link Node always requires a valid target.

## Extending

{% hint style="info" %}
Link vs. SubGraph: If the fragment is only needed within this asset, use [SubGraph](sub-graph.md) instead — no separate asset needed, easier to navigate in the editor.
{% endhint %}
