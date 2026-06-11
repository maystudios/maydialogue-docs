---
description: How MayDialogue behaves on dedicated and listen servers — the server-authoritative model, the client RPC flow, and what runs where.
---

# Multiplayer & Dedicated Server

MayDialogue is **server-authoritative**. The dialogue itself — the state machine, the node traversal, the variable writes, the choice evaluation — runs only on the server. Clients never run a parallel copy. Instead, they send their input *up* to the server and receive display data *down* from it.

If you have ever shipped a system that "works in PIE single-player but breaks on a dedicated server," this is the page that prevents it. The rules are simple once you internalize one fact: **`UMayDialogueInstance` lives on the server and has no client mirror object.**

{% hint style="info" %}
**Single-player projects can skip this page.** In Standalone everything runs locally and the authority checks below are no-ops. You only need this if you build for **listen server** (a player hosts) or **dedicated server**.
{% endhint %}

---

## The Core Rule: Server-Authoritative Instance

```text
SERVER                                   CLIENT (owning player)
──────                                   ──────────────────────
UMayDialogueSubsystem                    (no subsystem instance work)
  └─ ActiveDialogues[]                   ┌─ no UMayDialogueInstance
       └─ UMayDialogueInstance  ◀──RPC───┤    (server-only, not replicated)
            • runs the graph              │
            • writes variables            └─ UMayDialogueParticipant
            • evaluates choices                 • receives Client RPCs
            • broadcasts lifecycle              • sends Server RPCs
                  │                             • holds replicated mirror state
                  └────────RPC──────────────────▶ ClientUpdateConversation / ...
```

- `UMayDialogueInstance` is a plain `UObject` with **no replicated properties** and **no client-side counterpart**. It is created by `UMayDialogueSubsystem::StartDialogue`, which **rejects any call made on `NM_Client`** with an explanatory error.
- The network channel is the **`UMayDialogueParticipant`** actor component, modeled on Lyra's `UConversationParticipantComponent`. The participant carries the RPCs and the replicated mirror state that the client UI reads.

{% hint style="warning" %}
**Never call `StartDialogue` / `AdvanceDialogue` / `SelectChoice` / `AbortDialogue` directly on a client.** `UMayDialogueSubsystem::StartDialogue` fails loud on `NM_Client`:

> *"...called on a client; dialogues are server-authoritative. Trigger a Server RPC on your own actor and call StartDialogue from there."*

Route everything through the participant's net-safe `Request*` wrappers instead (below).
{% endhint %}

---

## The Client Flow (No-Code Path)

You do **not** write any RPCs yourself. The participant already carries them. For a client to drive a dialogue, wire your UI / input to the **net-safe `Request*` methods** — they work identically on a listen-server host, a dedicated-server client, and in Standalone:

| Wire your input to… | Forwards to (Server RPC) | Effect on the authoritative instance |
|---|---|---|
| `RequestStartDialogue(Target)` | `ServerStartConversation(Target)` | `UMayDialogueSubsystem::StartDialogue` — owner is Instigator |
| `RequestAdvanceDialogue()` | `ServerAdvanceConversation()` | `UMayDialogueInstance::AdvanceDialogue` |
| `RequestSelectChoice(Index)` | `ServerSelectChoice(Index)` | `UMayDialogueInstance::SelectChoice` (bounds-checked) |
| `RequestAbortDialogue()` | `ServerAbortConversation()` | `UMayDialogueInstance::AbortDialogue` |

In the other direction, the server pushes display data to the owning client via **Client RPCs**, which broadcast the delegates your UMG / Slate widgets are already bound to:

| Server calls (Client RPC) | Broadcasts | UI consumer |
|---|---|---|
| `ClientStartConversation()` | `OnDialogueStartedDelegate` (+ native) | Spawn / show the dialogue widget |
| `ClientExitConversation()` | `OnDialogueEndedDelegate` (+ native) | Tear down the widget |
| `ClientUpdateConversation(Message)` | `OnMessageReceived` (+ native) | Display the new line |
| `ClientUpdateConversationTaskChoiceData(Choices, PromptText)` | `OnChoicesReceived` (+ native) | Display the choice list |

> 📸 **Image placeholder:** `mp-client-flow-bp.png` — BP graph of a multiplayer-safe interaction on the player character.
> *Setup:* Player Character Blueprint. Graph: `On Interact Input Action` → `Get Component by Class (MayDialogueParticipant)` (Target: Self) → `Request Start Dialogue` (Target: NPC Actor Reference). Below, a separate strip: a UMG button's `OnClicked` → `Request Select Choice` (Choice Index from the button). Both nodes flagged with a comment box "Net-safe — works on client and listen-server host."

{% hint style="info" %}
Because the `Request*` wrappers forward through Server RPCs, you can wire a UMG button's `OnClicked` straight to `RequestSelectChoice` without any authority check — the underlying Server RPC performs the authority gate on the server.
{% endhint %}

---

## What Runs Where — The Net-Mode Matrix

This is the heart of the page: for each net mode, what the dialogue does and does **not** do.

| Aspect | Standalone | Listen-server **host** | Listen-server **remote client** | Dedicated server | DS **client** |
|---|---|---|---|---|---|
| Runs the `UMayDialogueInstance` (graph, variables, choices) | ✅ | ✅ | ❌ (server does) | ✅ | ❌ (server does) |
| Auto-spawns the dialogue **widget** | ✅ | ✅ | ✅ | ❌ (no viewport) | ✅ |
| Plays **audio / voice / Babel** | ✅ | ✅ | ✅ | ❌ (no audio device) | ✅ |
| Runs **camera** side effects (CameraFocus / CameraShake) | ✅ | ✅ | ✅ | ❌ (no local player) | ✅ |
| Receives lifecycle / message / choice data | direct | direct | via Client RPC + mirror | n/a | via Client RPC |
| Sends advance / choice / abort input | direct | direct | via Server RPC | n/a | via Server RPC |

### Dedicated server: the `NM_DedicatedServer` guards

A dedicated server has no `GameViewportClient`, no audio device, and no local `PlayerController`. MayDialogue guards all client-side work behind explicit `NM_DedicatedServer` checks so the server never asserts or null-derefs:

- **Widget auto-spawn** is skipped entirely in `StartDialogue` when `GetNetMode() == NM_DedicatedServer` — the Slate and UMG auto-spawn paths are both no-ops there.
- **Client effects** (audio, camera, Babel synth) are skipped: the traversal calls `Node->ExecuteClientEffects` only when the world is **not** `NM_DedicatedServer`.

The graph still runs to completion on the server — variables are written, choices resolve, lifecycle cues fire — it just produces no local presentation.

### Listen-server: host vs. remote owner

On a listen server, one machine is both server and a player:

- The **local host's own participant** uses the direct `OnRep` / delegate path — no RPC round-trip to itself.
- **Remote clients'** participants receive their Client RPC updates correctly. A per-participant connection check routes each Client RPC only to remote-owned participants, so the host does not double-deliver to itself.

{% hint style="info" %}
**Fixed 2026-06-06 (listen-server remote-owner delivery):** remote clients on a listen server now receive their Client RPC lifecycle/message/choice updates correctly. Earlier revisions could drop these on the remote-owner path; if you are on an older build and remote co-op players see no dialogue text, update.
{% endhint %}

---

## Client-Side Side Effects

Most side effects run server-side (`ExecuteSideEffect`) because they mutate authoritative state. Two are **cosmetic** and run client-side via `ExecuteClientSideEffect`:

| Side effect | Runs via | On dedicated server |
|---|---|---|
| `CameraFocus` (SideEffect form) | `ExecuteClientSideEffect` | No-op |
| `CameraShake` (SideEffect form) | `ExecuteClientSideEffect` | No-op |

`ExecuteClientSideEffect` is the cosmetic-only hook on `UMayDialogueSideEffect`, called separately from `ExecuteSideEffect`. If you author a **custom** side effect that does anything visual (a sound, a particle, a UI flash), put that code in `ExecuteClientSideEffect` — not `ExecuteSideEffect` — so it does not run on a headless server.

{% hint style="info" %}
The SideEffect form of CameraFocus supports SpeakerLook / Anchor modes but **not** Sequence mode — use the CameraFocus **Action-Node** if you need a Level Sequence.
{% endhint %}

---

## Voice Asset Streaming on Clients

When a SayLine plays in multiplayer, the voice line must be available on the **client** that hears it — the server has no audio device. The message struct the server replicates to clients (`FMayDialogueMessage`) carries the resolved voice as a **`TSoftObjectPtr<USoundBase>`**, exactly mirroring how it carries the speaker portrait. Because a soft pointer serializes as an `FSoftObjectPath`, the reference survives the Client RPC even when the underlying sound is not yet loaded on the receiving client — it never silently null-resolves on the wire.

```text
SERVER                              CLIENT
──────                              ──────
SayLine executes                    ClientUpdateConversation(Message)
  └─ server resolves the voice           └─ Message.Voice.IsNull()? → no voice, Babel may fill in
     (already loaded) and writes         └─ Voice already resident? → Voice.Get() → play
     it into Message.Voice as a          └─ not yet resident? → StreamableManager
     soft ref                                 async-load → on complete → play
```

How the widgets consume it (`SMayDialogueWidget` and `UMayDialogueWidget`):

- On the **server / listen-server host / standalone**, the server set `Message.Voice` from an already-loaded asset, so the fast path `Voice.Get()` returns it immediately.
- On a **remote client**, the widget async-loads via `UAssetManager::GetStreamableManager().RequestAsyncLoad(Voice.ToSoftObjectPath())` — the same pattern it uses for the speaker portrait — then plays on completion.
- A **non-null** soft path means the line *has* a real voice asset (resident or not), so the Babel fallback is correctly skipped; `Voice.IsNull()` is what distinguishes "no voice authored" from "voice not yet loaded".

{% hint style="info" %}
`FMayDialogueMessage` is a runtime-only transport struct (a Client-RPC argument and the replicated `UMayDialogueParticipant::CurrentMessage` mirror) — it is never serialized into a saved `.uasset`, so this soft-ref design needs no asset migration.
{% endhint %}

---

## Known Limitations (Honest List)

MayDialogue ships a **server-authoritative, single-driver** dialogue. The following are **intentional non-goals**, not bugs:

- **No choice voting.** A dialogue is driven by one authoritative input path. There is no built-in mechanism for multiple players to vote on a choice.
- **No shared / co-op dialogue sessions.** One participant drives; other clients can *watch* (via the replicated mirror state on non-owner participants) but there is no shared-control session model.
- **No multiplayer UX layer.** Shared sessions, voting UI, and turn arbitration are out of scope — they belong in a project-specific layer on top of the bridge.

If your game needs voting or shared control, build it on top: the server owns the instance, so your custom Server RPC can call `SelectChoice` after *your* arbitration logic decides the winning choice.

> 📸 **Image placeholder:** `mp-spectator-mirror.png` — Two PIE client windows: one driving a dialogue, one spectating.
> *Setup:* PIE with 2 clients (Net Mode: Play As Listen Server, 2 players). Client 1 is mid-dialogue with an NPC, choice list visible. Client 2 is positioned to watch the same NPC and shows the same current message text (read from the replicated `CurrentMessage` mirror on the non-owner participant) but **no** interactive choice buttons. Caption-style annotation pointing at Client 2: "non-owner — sees state via mirror, cannot drive."

---

## See Also

- [Starting a Dialogue](starting-dialogues.md) — the `RequestStartDialogue` vs. `StartDefaultDialogue` role table.
- [Subsystem API](subsystem-api.md) — `StartDialogue`, `AbortDialogue`, `AbortAllDialogues` (all server/authority).
- [Bridge & Lifecycle Events](bridge-events.md) — the delegates the Client RPCs broadcast.
