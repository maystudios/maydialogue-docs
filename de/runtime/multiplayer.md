---
description: Wie sich MayDialogue auf Dedicated- und Listen-Servern verhält — das server-autoritative Modell, der Client-RPC-Fluss und was wo läuft.
---

# Multiplayer & Dedicated Server

MayDialogue ist **server-autoritativ**. Der Dialog selbst — die State-Machine, die Node-Traversierung, die Variablen-Schreibvorgänge, die Choice-Auswertung — läuft nur auf dem Server. Clients führen keine parallele Kopie aus. Stattdessen schicken sie ihren Input *nach oben* zum Server und erhalten Anzeige-Daten *nach unten* von ihm.

Wenn du je ein System ausgeliefert hast, das "im PIE-Einzelspieler funktioniert, aber auf einem Dedicated Server bricht", ist das die Seite, die das verhindert. Die Regeln sind einfach, sobald du eine Tatsache verinnerlicht hast: **`UMayDialogueInstance` lebt auf dem Server und hat kein Client-Spiegel-Objekt.**

{% hint style="info" %}
**Einzelspieler-Projekte können diese Seite überspringen.** In Standalone läuft alles lokal und die Authority-Checks unten sind No-Ops. Du brauchst sie nur, wenn du für **Listen-Server** (ein Spieler hostet) oder **Dedicated Server** baust.
{% endhint %}

---

## Die Kernregel: Server-autoritative Instance

```text
SERVER                                   CLIENT (besitzender Spieler)
──────                                   ───────────────────────────
UMayDialogueSubsystem                    (keine Subsystem-Instance-Arbeit)
  └─ ActiveDialogues[]                   ┌─ keine UMayDialogueInstance
       └─ UMayDialogueInstance  ◀──RPC───┤    (nur Server, nicht repliziert)
            • führt den Graph aus         │
            • schreibt Variablen          └─ UMayDialogueParticipant
            • wertet Choices aus               • empfängt Client-RPCs
            • broadcastet Lifecycle            • sendet Server-RPCs
                  │                            • hält replizierten Mirror-State
                  └────────RPC──────────────────▶ ClientUpdateConversation / ...
```

- `UMayDialogueInstance` ist ein einfaches `UObject` mit **keinen replizierten Properties** und **keinem Client-seitigen Gegenstück**. Es wird von `UMayDialogueSubsystem::StartDialogue` erzeugt, das **jeden auf `NM_Client` gemachten Aufruf** mit einem erklärenden Error **ablehnt**.
- Der Netzwerk-Kanal ist die **`UMayDialogueParticipant`**-Actor-Komponente, modelliert nach Lyras `UConversationParticipantComponent`. Der Participant trägt die RPCs und den replizierten Mirror-State, den die Client-UI liest.

{% hint style="warning" %}
**Rufe `StartDialogue` / `AdvanceDialogue` / `SelectChoice` / `AbortDialogue` niemals direkt auf einem Client.** `UMayDialogueSubsystem::StartDialogue` schlägt auf `NM_Client` laut fehl:

> *"...called on a client; dialogues are server-authoritative. Trigger a Server RPC on your own actor and call StartDialogue from there."*

Leite stattdessen alles über die netz-sicheren `Request*`-Wrapper des Participants (unten).
{% endhint %}

---

## Der Client-Fluss (No-Code-Weg)

Du schreibst **keine** RPCs selbst. Der Participant trägt sie bereits. Damit ein Client einen Dialog steuert, verdrahtest du deine UI / deinen Input mit den **netz-sicheren `Request*`-Methoden** — sie funktionieren identisch auf einem Listen-Server-Host, einem Dedicated-Server-Client und in Standalone:

| Verdrahte deinen Input mit… | Leitet weiter an (Server-RPC) | Effekt auf die autoritative Instance |
|---|---|---|
| `RequestStartDialogue(Target)` | `ServerStartConversation(Target)` | `UMayDialogueSubsystem::StartDialogue` — Eigentümer ist Instigator |
| `RequestAdvanceDialogue()` | `ServerAdvanceConversation()` | `UMayDialogueInstance::AdvanceDialogue` |
| `RequestSelectChoice(Index)` | `ServerSelectChoice(Index)` | `UMayDialogueInstance::SelectChoice` (Bounds-geprüft) |
| `RequestAbortDialogue()` | `ServerAbortConversation()` | `UMayDialogueInstance::AbortDialogue` |

In die andere Richtung schiebt der Server Anzeige-Daten zum besitzenden Client über **Client-RPCs**, die die Delegates broadcasten, an die deine UMG-/Slate-Widgets ohnehin schon gebunden sind:

| Server ruft (Client-RPC) | Broadcastet | UI-Konsument |
|---|---|---|
| `ClientStartConversation()` | `OnDialogueStartedDelegate` (+ native) | Dialog-Widget spawnen / zeigen |
| `ClientExitConversation()` | `OnDialogueEndedDelegate` (+ native) | Widget abbauen |
| `ClientUpdateConversation(Message)` | `OnMessageReceived` (+ native) | Neue Zeile anzeigen |
| `ClientUpdateConversationTaskChoiceData(Choices, PromptText)` | `OnChoicesReceived` (+ native) | Choice-Liste anzeigen |

> 📸 **Bild-Platzhalter:** `mp-client-flow-bp.png` — BP-Graph einer multiplayer-sicheren Interaktion am Spieler-Charakter.
> *Setup:* Spieler-Charakter-Blueprint. Graph: `On Interact Input Action` → `Get Component by Class (MayDialogueParticipant)` (Target: Self) → `Request Start Dialogue` (Target: NPC-Actor-Referenz). Darunter ein separater Strang: das `OnClicked` eines UMG-Buttons → `Request Select Choice` (Choice Index vom Button). Beide Nodes mit einer Kommentar-Box "Netz-sicher — funktioniert auf Client und Listen-Server-Host" markiert.

{% hint style="info" %}
Weil die `Request*`-Wrapper über Server-RPCs weiterleiten, kannst du das `OnClicked` eines UMG-Buttons direkt mit `RequestSelectChoice` verdrahten — ohne jeden Authority-Check. Der zugrunde liegende Server-RPC führt das Authority-Gate auf dem Server aus.
{% endhint %}

---

## Was wo läuft — die Net-Mode-Matrix

Das ist der Kern der Seite: Für jeden Net-Mode, was der Dialog tut und **nicht** tut.

| Aspekt | Standalone | Listen-Server **Host** | Listen-Server **Remote-Client** | Dedicated Server | DS-**Client** |
|---|---|---|---|---|---|
| Führt die `UMayDialogueInstance` aus (Graph, Variablen, Choices) | ✅ | ✅ | ❌ (Server tut es) | ✅ | ❌ (Server tut es) |
| Auto-spawnt das Dialog-**Widget** | ✅ | ✅ | ✅ | ❌ (kein Viewport) | ✅ |
| Spielt **Audio / Voice / Babel** | ✅ | ✅ | ✅ | ❌ (kein Audio-Gerät) | ✅ |
| Führt **Kamera**-Side-Effects aus (CameraFocus / CameraShake) | ✅ | ✅ | ✅ | ❌ (kein lokaler Spieler) | ✅ |
| Empfängt Lifecycle-/Message-/Choice-Daten | direkt | direkt | via Client-RPC + Mirror | n/v | via Client-RPC |
| Sendet Advance-/Choice-/Abort-Input | direkt | direkt | via Server-RPC | n/v | via Server-RPC |

### Dedicated Server: die `NM_DedicatedServer`-Guards

Ein Dedicated Server hat keinen `GameViewportClient`, kein Audio-Gerät und keinen lokalen `PlayerController`. MayDialogue schützt alle Client-seitige Arbeit hinter expliziten `NM_DedicatedServer`-Checks, damit der Server niemals asserted oder null-dereferenziert:

- **Widget-Auto-Spawn** wird in `StartDialogue` komplett übersprungen, wenn `GetNetMode() == NM_DedicatedServer` — die Slate- und UMG-Auto-Spawn-Pfade sind dort beide No-Ops.
- **Client-Effekte** (Audio, Kamera, Babel-Synth) werden übersprungen: die Traversierung ruft `Node->ExecuteClientEffects` nur, wenn die Welt **nicht** `NM_DedicatedServer` ist.

Der Graph läuft auf dem Server trotzdem vollständig durch — Variablen werden geschrieben, Choices aufgelöst, Lifecycle-Cues feuern — er erzeugt nur keine lokale Darstellung.

### Listen-Server: Host vs. Remote-Owner

Auf einem Listen-Server ist eine Maschine sowohl Server als auch Spieler:

- Der **eigene Participant des lokalen Hosts** nutzt den direkten `OnRep`-/Delegate-Pfad — kein RPC-Roundtrip zu sich selbst.
- Die Participants der **Remote-Clients** empfangen ihre Client-RPC-Updates korrekt. Ein Per-Participant-Connection-Check leitet jeden Client-RPC nur an remote-besessene Participants, sodass der Host nicht doppelt an sich selbst ausliefert.

{% hint style="info" %}
**Behoben 2026-06-06 (Listen-Server-Remote-Owner-Zustellung):** Remote-Clients auf einem Listen-Server empfangen ihre Client-RPC-Lifecycle-/Message-/Choice-Updates jetzt korrekt. Frühere Revisionen konnten diese auf dem Remote-Owner-Pfad verlieren; wenn du auf einem älteren Build bist und Remote-Koop-Spieler keinen Dialogtext sehen, aktualisiere.
{% endhint %}

---

## Client-seitige Side-Effects

Die meisten Side-Effects laufen server-seitig (`ExecuteSideEffect`), weil sie autoritativen State mutieren. Zwei sind **kosmetisch** und laufen client-seitig über `ExecuteClientSideEffect`:

| Side-Effect | Läuft über | Auf Dedicated Server |
|---|---|---|
| `CameraFocus` (SideEffect-Form) | `ExecuteClientSideEffect` | No-Op |
| `CameraShake` (SideEffect-Form) | `ExecuteClientSideEffect` | No-Op |

`ExecuteClientSideEffect` ist der kosmetik-only-Hook auf `UMayDialogueSideEffect`, getrennt von `ExecuteSideEffect` aufgerufen. Wenn du einen **eigenen** Side-Effect schreibst, der irgendetwas Visuelles tut (ein Sound, ein Partikel, ein UI-Flash), packe diesen Code in `ExecuteClientSideEffect` — nicht in `ExecuteSideEffect` — damit er nicht auf einem Headless-Server läuft.

{% hint style="info" %}
Die SideEffect-Form von CameraFocus unterstützt SpeakerLook-/Anchor-Modi, aber **nicht** den Sequence-Modus — nutze die CameraFocus-**Action-Node**, wenn du eine Level Sequence brauchst.
{% endhint %}

---

## Voice-Asset-Streaming auf Clients

Wenn eine SayLine im Multiplayer spielt, muss die Sprachzeile auf dem **Client** verfügbar sein, der sie hört — der Server hat kein Audio-Gerät. Das Message-Struct, das der Server an Clients repliziert (`FMayDialogueMessage`), trägt die aufgelöste Voice als **`TSoftObjectPtr<USoundBase>`** — exakt so, wie es das Speaker-Portrait trägt. Weil ein Soft-Pointer als `FSoftObjectPath` serialisiert, überlebt die Referenz den Client-RPC auch dann, wenn der zugrunde liegende Sound auf dem empfangenden Client noch nicht geladen ist — sie löst auf der Leitung nie still zu null auf.

```text
SERVER                              CLIENT
──────                              ──────
SayLine läuft                       ClientUpdateConversation(Message)
  └─ Server löst die Voice auf          └─ Message.Voice.IsNull()? → keine Voice, Babel kann einspringen
     (bereits geladen) und schreibt     └─ Voice schon resident? → Voice.Get() → spielen
     sie als Soft-Ref in Message.Voice  └─ noch nicht resident? → StreamableManager
                                             async laden → bei Complete → spielen
```

Wie die Widgets es konsumieren (`SMayDialogueWidget` und `UMayDialogueWidget`):

- Auf dem **Server / Listen-Server-Host / Standalone** hat der Server `Message.Voice` aus einem bereits geladenen Asset gesetzt, sodass der Fast-Path `Voice.Get()` es sofort zurückgibt.
- Auf einem **Remote-Client** lädt das Widget asynchron über `UAssetManager::GetStreamableManager().RequestAsyncLoad(Voice.ToSoftObjectPath())` — dasselbe Muster, das es für das Speaker-Portrait nutzt — und spielt dann bei Completion.
- Ein **nicht-null**er Soft-Pfad bedeutet, dass die Zeile *ein* echtes Voice-Asset hat (resident oder nicht), sodass der Babel-Fallback korrekt übersprungen wird; `Voice.IsNull()` ist das, was „keine Voice gesetzt" von „Voice noch nicht geladen" unterscheidet.

{% hint style="info" %}
`FMayDialogueMessage` ist ein reines Runtime-Transport-Struct (ein Client-RPC-Argument und der replizierte `UMayDialogueParticipant::CurrentMessage`-Mirror) — es wird nie in ein gespeichertes `.uasset` serialisiert, sodass dieses Soft-Ref-Design keine Asset-Migration braucht.
{% endhint %}

---

## Bekannte Einschränkungen (ehrliche Liste)

MayDialogue liefert einen **server-autoritativen Single-Driver**-Dialog. Die folgenden Punkte sind **bewusste Nicht-Ziele**, keine Bugs:

- **Kein Choice-Voting.** Ein Dialog wird von einem autoritativen Input-Pfad getrieben. Es gibt keinen eingebauten Mechanismus, mit dem mehrere Spieler über eine Choice abstimmen.
- **Keine geteilten / Koop-Dialog-Sessions.** Ein Participant treibt; andere Clients können *zusehen* (über den replizierten Mirror-State auf Nicht-Eigentümer-Participants), aber es gibt kein Shared-Control-Session-Modell.
- **Keine Multiplayer-UX-Schicht.** Geteilte Sessions, Voting-UI und Turn-Arbitrierung sind außerhalb des Scopes — sie gehören in eine projektspezifische Schicht über der Bridge.

Wenn dein Spiel Voting oder geteilte Steuerung braucht, bau es obendrauf: Der Server besitzt die Instance, also kann dein eigener Server-RPC `SelectChoice` aufrufen, nachdem *deine* Arbitrierungslogik die Gewinner-Choice entschieden hat.

> 📸 **Bild-Platzhalter:** `mp-spectator-mirror.png` — Zwei PIE-Client-Fenster: eines treibt einen Dialog, eines schaut zu.
> *Setup:* PIE mit 2 Clients (Net Mode: Play As Listen Server, 2 Spieler). Client 1 ist mitten im Dialog mit einem NPC, Choice-Liste sichtbar. Client 2 steht so, dass er denselben NPC beobachtet, und zeigt denselben aktuellen Message-Text (gelesen aus dem replizierten `CurrentMessage`-Mirror auf dem Nicht-Eigentümer-Participant), aber **keine** interaktiven Choice-Buttons. Beschriftung mit Pfeil auf Client 2: "Nicht-Eigentümer — sieht State via Mirror, kann nicht treiben."

---

## Siehe auch

- [Einen Dialog starten](starting-dialogues.md) — die Rollen-Tabelle `RequestStartDialogue` vs. `StartDefaultDialogue`.
- [Subsystem-API](subsystem-api.md) — `StartDialogue`, `AbortDialogue`, `AbortAllDialogues` (alle Server/Authority).
- [Bridge & Lifecycle-Events](bridge-events.md) — die Delegates, die die Client-RPCs broadcasten.
