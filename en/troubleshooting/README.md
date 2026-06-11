---
description: Quickly find the right chapter when something isn't working.
---

# Troubleshooting — Guide

This section answers the most common questions when a dialogue doesn't behave as expected. Read through the diagnostic questions below first — in most cases you'll find the answer in under five minutes.

## What Do You Want to Fix?

| Symptom | Page |
| --- | --- |
| Dialogue doesn't start at all, nothing happens | [Common Issues → Dialogue doesn't start](common-issues.md#dialogue-doesnt-start) |
| No widget visible even though the dialogue is running | [Common Issues → Widget doesn't appear](common-issues.md#widget-doesnt-appear) |
| Choices are missing or not clickable | [Common Issues → Choices missing](common-issues.md#choices-missing) |
| No audio / no voice | [Common Issues → Audio missing](common-issues.md#audio-missing) |
| Babel voice is silent | [Common Issues → Babel silent](common-issues.md#babel-silent) |
| Variable isn't saved / Requirement reads wrong values | [Common Issues → Variable not persistent](common-issues.md#variable-not-persistent) |
| Dialogue gets stuck after one line | [Common Issues → Dialogue stuck](common-issues.md#dialogue-gets-stuck) |
| Compile error with no clear hint | [Common Issues → Compile error](common-issues.md#compile-error) |
| Sub-Graph doesn't return to caller | [Common Issues → Sub-Graph return missing](common-issues.md#sub-graph-return-missing) |
| Log error or crash | [Debugging Tips](debugging-tips.md) |
| Known issue, looking for a workaround | [Known Issues](known-issues.md) |

## Recommended Debug Flow

```
Dialogue behaves incorrectly
  │
  ├─ Does it run correctly in the Preview Runner?
  │     │
  │     ├─ No → Structural error in the asset (nodes, links, requirements)
  │     │          → Check common-issues.md + validator messages
  │     │
  │     └─ Yes → Problem is outside the asset
  │               ├─ Widget not visible → Check widget setup
  │               ├─ Participant error → Check participant component + tags
  │               └─ GAS dependency → Check ASC / tag state
  │
  └─ Debugger + Output Log → debugging-tips.md
```

{% hint style="info" %}
**Fastest starting point:** Open the asset and click **Compile**. Many problems are compile errors that the validator identifies immediately.
{% endhint %}

## Structure of This Section

- **[Common Issues](common-issues.md)** — Per symptom: cause + solution.
- **[Debugging Tips](debugging-tips.md)** — Output Log, debugger, Preview Runner, isolation tests.
- **[Known Issues](known-issues.md)** — Current limitations and recommended approaches.
