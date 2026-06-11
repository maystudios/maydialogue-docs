---
description: How MayDialogue spends frame time, how to measure it, and where the hitch risks are.
---

# Performance

MayDialogue is built to be cheap at idle and predictable under load. A dialogue is a synchronous state machine that only does work when something happens — a node executes, a choice is evaluated, a voice plays. Between those moments it costs almost nothing.

This page shows you **how to see** where the time goes (`stat MayDialogue`, Unreal Insights), explains the **async pre-warm** that hides the few genuinely expensive operations, and lists the **known hitch-risk points** so you can plan around them.

{% hint style="info" %}
**Who needs this page?** Most projects never have to think about dialogue performance — a single conversation is a rounding error in a frame budget. Read this if you run **many concurrent dialogues** (ambient NPC chatter across a town), if you see a **hitch when a line plays**, or if you are doing a profiling pass before ship.
{% endhint %}

---

## The No-Code Path: `stat MayDialogue`

Every meaningful piece of dialogue work is wrapped in a named counter. To see them live:

1. Play in editor (PIE) or run a packaged build.
2. Open the console (`~`) and type:

   ```text
   stat MayDialogue
   ```

3. Start a dialogue. The on-screen overlay now shows per-frame timings for each counter, with min/max/average.

To turn it off, type `stat MayDialogue` again.

> 📸 **Image placeholder:** `stat-maydialogue-overlay.png` — The `stat MayDialogue` overlay running during a dialogue.
> *Setup:* PIE session with a dialogue actively running (a SayLine with Babel voice on screen). Console command `stat MayDialogue` entered. The stat overlay is visible in the top-left, showing the `ContinueToNode`, `ExecuteNode`, and `SayLine_*` rows with non-zero values. Frame counter visible. Crop tight to the overlay so the row names are legible.

---

## The Instrumented Counters

All counters live in the **`MayDialogue` stat group** (`STATGROUP_MayDialogue`), declared in `MayDialogueStats.h`. Each one is both a `stat` cycle counter (visible via `stat MayDialogue`) and an Unreal Insights CPU trace scope (visible in Insights under the `MayDialogue::` namespace — see [Unreal Insights](#unreal-insights-deep-profiling) below).

### Hot-path counters (confirmed in source)

| Counter (`stat` name) | Wraps | When it fires |
|---|---|---|
| **ContinueToNode** | `UMayDialogueInstance::ContinueToNode` | The full traversal loop — every burst of node-to-node advancement. The top-level dialogue cost. |
| **ExecuteNode** | A single node dispatch (`ExecuteNode`) inside the loop | Once per node executed. |
| **RefreshPendingChoices** | `UMayDialogueInstance::RefreshPendingChoices` | Whenever staged choices are re-evaluated (e.g. a variable changed while waiting on a choice). |
| **SayLine_ResolveAudio** | `ResolveAudioSettings` — the four-level audio chain | Once per SayLine that plays audio. |
| **SayLine_ResolveVoice** | `ResolveVoice` — voice asset + per-culture lookup | Once per SayLine. |
| **SayLine_BabelFallback** | `ExecuteBabelFallback` — Babel profile resolve + synth spawn | Only when a SayLine has no voice asset and Babel is enabled. |
| **CameraFocus_Sequence** | `UMayDialogueNode_CameraFocus::ExecuteClientEffects` — Level Sequence load + spawn | Only on a CameraFocus node in Sequence mode. **Hitch risk** — see below. |
| **Link_LoadTarget** | `UMayDialogueNode_Link::ExecuteNode` — linked asset synchronous load | Only on a Link node whose target asset is not yet in memory. **Hitch risk** — see below. |
| **StartDialogue_PreWarm** | `UMayDialogueInstance::PreWarmAssets` — async soft-ref gather at start | Once per `StartDialogue`. Cheap — it only *requests* loads, it does not block. |

### Per-frame tick & gauge counters

The counters above are **event-driven** — they only fire when a node runs. The following counters measure **continuous per-frame cost** and the live instance count, so you can spot an idle leak (a dialogue that ticks forever) or a concurrency spike:

| Counter (`stat` name) | Wraps | Reads as |
|---|---|---|
| **SubsystemTick** | `UMayDialogueSubsystem::Tick` | Per-frame cost of the subsystem fan-out across all active instances. Zero frames where no dialogue is active (the subsystem only ticks while `ActiveDialogues.Num() > 0`). |
| **InstanceTick** | `UMayDialogueInstance::Tick` | Per-frame cost of a single instance's tick (auto-advance timers, choice timeouts, focus-camera tracking). |
| **BabelSynthTick** | `UMayDialogueBabelSynth::Tick` | Per-frame cost of one active Babel synth voice (Continuous sync-mode driver). |
| **ActiveDialogues** (gauge) | The subsystem's `ActiveDialogues` array length | A *count*, not a time — how many dialogues are running concurrently this frame. |

{% hint style="info" %}
The four tick/gauge counters in the table directly above (`SubsystemTick`, `InstanceTick`, `BabelSynthTick`, and the `ActiveDialogues` gauge) are **present in source now** — they are declared in `MayDialogueStats.h` alongside the hot-path counters. The functions they wrap (`UMayDialogueSubsystem::Tick`, `UMayDialogueInstance::Tick`, `UMayDialogueBabelSynth::Tick`) tick today. The representative *measurement numbers* in the table at the bottom of this page come from the 1.0 perf pass (2026-06).
{% endhint %}

---

## Unreal Insights (Deep Profiling)

`stat MayDialogue` gives you per-frame averages on screen. For a frame-by-frame timeline — exactly which node took which microseconds, and how it overlapped with the rest of your game — use **Unreal Insights**.

Every MayDialogue counter is also a CPU trace scope under the `MayDialogue::` namespace (e.g. `MayDialogue::ContinueToNode`, `MayDialogue::ExecuteNode`, `MayDialogue::StartDialogue_PreWarm`). To capture them:

1. Launch your game with CPU tracing enabled:

   ```text
   -trace=cpu,frame,bookmark
   ```

2. Reproduce the dialogue scene you want to profile.
3. Open the resulting `.utrace` in Unreal Insights and filter the timing tracks for `MayDialogue`.

> 📸 **Image placeholder:** `insights-maydialogue-trace.png` — Unreal Insights timeline filtered to MayDialogue scopes.
> *Setup:* Unreal Insights open on a `.utrace` captured with `-trace=cpu,frame`. The Timing Insights view is filtered (search box) to `MayDialogue`. Visible scopes: `MayDialogue::ContinueToNode` containing nested `MayDialogue::ExecuteNode` blocks, with a `MayDialogue::SayLine_ResolveVoice` child. Pick a frame where a SayLine fired so the nesting is clear.

---

## How Pre-Warm Hides the Expensive Parts

Three operations in a dialogue are genuinely expensive because they touch disk: loading an **attenuation / sound-class / Babel-profile** asset for audio, loading a **Level Sequence** for a CameraFocus, and loading a **linked dialogue asset** for a Link node. Done synchronously at the moment of use, any of these can cause a frame hitch.

To hide that cost, `UMayDialogueInstance::StartDialogue` calls **`PreWarmAssets`** before the first node ever runs. It walks every compiled node in the asset, gathers each node's soft-referenced assets, and fires a **single fire-and-forget async load** through the StreamableManager:

```text
StartDialogue
    └─> PreWarmAssets(Asset)            // STAT: StartDialogue_PreWarm
          ├─ gather plugin-default soft-refs (attenuation, sound class, Babel profile)
          ├─ walk CompiledNodes:
          │     SayLine    → speaker attenuation / sound-class / Babel profile
          │     CameraFocus → Level Sequence
          │     Link        → target dialogue asset
          └─ StreamableManager::RequestAsyncLoad(all paths)   // async, non-blocking
```

By the time the player reaches a SayLine, CameraFocus, or Link node — usually several seconds of conversation later — the assets are already resident in memory. The per-node code still calls `LoadSynchronous` as a **correctness fallback**, but it finds the asset in cache and returns instantly instead of hitting disk.

{% hint style="info" %}
Pre-warm is **best-effort**, not a guarantee. If a player blitzes through a dialogue faster than the async load completes, or if the asset manager is unavailable (headless test environments), the synchronous fallback still runs and may hitch. For very large Level Sequences or deeply chained Link targets, see the gotchas below.
{% endhint %}

---

## Known Hitch-Risk Points

| Risk | Why | Mitigation |
|---|---|---|
| **CameraFocus in Sequence mode** | Loading + spawning a Level Sequence on the frame the node executes (`CameraFocus_Sequence`). Large sequences with many tracks are the worst case. | Pre-warm gathers the `CameraSequence` soft-ref at `StartDialogue`. Keep sequences lean; put the CameraFocus a few lines into the conversation so pre-warm has time. |
| **Link target load** | A Link node whose `TargetDialogueAsset` is not yet in memory loads it synchronously (`Link_LoadTarget`). | Pre-warm gathers the immediate Link target. Note: pre-warm only walks **one** asset's compiled nodes — a chain of Links into Links-into-Links only pre-warms the first hop. |
| **Babel synth spawn** | The first frame a Babel voice starts (`SayLine_BabelFallback`) allocates and configures the synth. | Cheap relative to a real voice asset, but it is per-line. If you have dense ambient Babel chatter, watch `BabelSynthTick` × concurrent-voice count. |
| **Many concurrent dialogues** | Every active dialogue ticks (`SubsystemTick` fans out to `InstanceTick`). | The cost scales linearly with `ActiveDialogues`. Watch the `ActiveDialogues` gauge; cap ambient conversations if you spawn dozens. |

---

## Profiling a Dialogue Scene — A Recipe

1. **Reproduce the worst case.** Set up the scene with the most concurrent dialogues / the heaviest CameraFocus you ship.
2. **Watch `stat MayDialogue`** live. Note which counter dominates. A spike in `CameraFocus_Sequence` or `Link_LoadTarget` points at a load that pre-warm did not cover (too fast, or a deep Link chain).
3. **Confirm with `stat Unit`** that the spike actually shows up in `Game` thread time — dialogue runs on the game thread.
4. **Capture an Insights trace** (`-trace=cpu,frame`) for a frame-exact view of the nesting and to see what *else* ran in the hitch frame.
5. **Check the gauge.** If `ActiveDialogues` is higher than you expect, you have dialogues that never ended — look for a Wait node with no resume, or an instance whose participant was destroyed without `EndPlay` firing.

---

## Measurements

Measured in the 1.0 perf pass (2026-06): UE 5.7, Win64 **Development editor PIE**, AMD Ryzen 9 9950X3D, sample map `L_DialogueShowcase` with sample dialogues. Per-frame values are `stat DumpAve` averages over 60–120 frames; the `StartDialogue` timings are in-process measurements around the subsystem call. A Shipping build without editor overhead tends to come in lower — treat these as orders of magnitude, not guarantees, and re-measure on your target hardware.

| What | Configuration | Cost | Notes |
|---|---|---|---|
| Subsystem tick — idle | No active dialogue | **0** (does not tick) | Confirmed: the counter does not appear in the idle dump at all — the subsystem does not tick while `ActiveDialogues.Num() == 0`. |
| Subsystem tick — active | 1 active dialogue, no async work this frame | **~0.004 ms** | `SubsystemTick` (includes the fan-out); plus one `InstanceTick`. |
| Instance tick — active | 1 instance, auto-advance/focus tracking running | **~0.001 ms** | `InstanceTick`. |
| Babel synth — per active voice | 1 Continuous-mode Babel voice | **~0.004 ms** | `BabelSynthTick`, per-voice CPU. In typewriter-sync mode (the default of every sample profile) the synth does not self-tick at all — 0. |
| Memory — per active dialogue | 1 `UMayDialogueInstance` + async state | **~2 KB + ~0.1 KB per AsyncState** | `obj list`: instance 1.74 KB (max 2.05 KB), one `MayDialogueNodeAsyncState` 0.11 KB. |
| Widget construct cost | First UMG widget auto-spawn at `StartDialogue` | **~4.6 ms** (one-time) | Delta of first vs. second start (8.4 − 3.8 ms). One-time per subsystem lifetime (reused after). |
| StartDialogue (cold) | First start in the level (incl. the one-time widget spawn) | **~8.4 ms** | Instance creation + `StartDialogue_PreWarm` request + first node + widget spawn. Disk loads for soft refs run asynchronously via pre-warm and are not part of this number. |
| StartDialogue (warm) | Widget up, asset resident | **~3.8 ms** | Second dialogue in the same level; pre-warm finds everything in cache. |
