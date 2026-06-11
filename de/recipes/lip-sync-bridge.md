---
description: Treibe jede Lip-Sync-Lösung aus den Dialog-Voice-Lines – binde die Voice-Line-Events, füttere deine Blendshapes oder Viseme, räume am Dialog-Ende auf.
---

# Lip-Sync anbinden (Bridge-Rezept)

## Szenario

Deine Charaktere sollen den Mund bewegen, während sie eine Dialog-Zeile sprechen. MayDialogue liefert **kein** Lip-Sync-System – das ist Absicht, es hält sich aus der Facial-Animation heraus. Stattdessen gibt es dir präzise Voice-Line-Events, mit denen du *deine* Lip-Sync-Lösung treibst: einen Audio-Amplituden-Blendshape-Treiber, ein Viseme-basiertes Tool (OVR LipSync, NVIDIA Audio2Face, MetaHuman) oder ein handgekeytes Mund-Flap.

Dieses Rezept ist das **Integrations-Muster**: `OnVoiceLineStarted` / `OnVoiceLineEnded` binden, deinen Treiber füttern und am Dialog-Ende abbauen.

## Was du lernst

- Die Voice-Line-Lifecycle-Events an der Dialog-Instanz binden.
- Das Voice-Asset der Zeile in einen amplituden- oder visemebasierten Lip-Sync-Treiber füttern.
- Lip-Sync sauber bei `OnVoiceLineEnded` und `OnDialogueEnded` stoppen.
- Den Multiplayer-Fall ehrlich behandeln (die Voice-Line-Events sind server-only).

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- Eine Lip-Sync-Lösung, die isoliert bereits funktioniert (du kannst einen Kopf reden lassen, gegeben ein `USoundBase` oder eine Audio-Amplitude). MayDialogue liefert den *Trigger*, nicht das Lip-Sync selbst.
- Der Sprecher-Actor hat ein Skeletal Mesh mit Mund-Blendshapes oder ein Viseme-Curve-Setup.

## Das Muster

```text
[OnDialogueStarted]
   └─▶ sprechenden Actor auflösen, Voice-Events binden
        │
        ├─[OnVoiceLineStarted (NodeGuid, SpeakerTag, Sound)]
        │     └─▶ LIP-SYNC AN: Sound (oder Amplitude) in deinen Treiber füttern
        │
        ├─[OnVoiceLineEnded   (NodeGuid, SpeakerTag, Sound)]
        │     └─▶ LIP-SYNC AUS: Mund schließen, Treiber stoppen
        │
        └─[OnDialogueEnded]
              └─▶ alles unbinden, laufendes Lip-Sync hart stoppen
```

> 📸 **Bild-Platzhalter:** `lip-sync-bridge-flow-diagram.png` — Der Bind → Drive → Cleanup-Flow.
> *Setup:* Eine Grafik erstellen (kein Editor-Screenshot). Oben Node `[OnDialogueStarted: bind]`, verzweigt nach unten in drei Boxen: `[OnVoiceLineStarted → LipSync AN]` (grün), `[OnVoiceLineEnded → LipSync AUS]` (rot), `[OnDialogueEnded → unbind + hart-stoppen]` (grau). Vertikales Layout, weißer Hintergrund.

## Schritt-für-Schritt (Blueprint)

### 1. Events binden, wenn der Dialog startet

Im Actor (oder Manager), dem der Sprecher gehört, an die Voice-Events der Instanz binden. Die Instanz kommt aus der `OnDialogueStarted`-Payload oder über `UMayDialogueSubsystem → Get Active Dialogue`.

```text
[OnDialogueStarted]
   │
   ▼
[Bind Event to OnVoiceLineStarted] ── HandleVoiceStart
[Bind Event to OnVoiceLineEnded]   ── HandleVoiceEnd
```

> 📸 **Bild-Platzhalter:** `lip-sync-bridge-bp-bind.png` — Blueprint bindet die zwei Voice-Line-Events.
> *Setup:* Blueprint-Event-Graph. Rotes Event-Node `OnDialogueStarted` → zwei `Bind Event to ...`-Nodes für `OnVoiceLineStarted` und `OnVoiceLineEnded`, jeweils mit einem Custom-Event verdrahtet ("HandleVoiceStart", "HandleVoiceEnd"). Sauberes Layout.

### 2. Lip-Sync bei Voice Line Started starten

`HandleVoiceStart` erhält `(NodeGuid, SpeakerTag, Sound)`:
- Den Actor für `SpeakerTag` auflösen (z.B. `Get Active Participant Actors` + Match, oder deine eigene Sprecher-Registry).
- Den `Sound` an deinen Lip-Sync-Treiber übergeben oder deinen Amplituden-Sampler gegen das aktive Voice-Audio starten.

```text
[HandleVoiceStart (SpeakerTag, Sound)]
   │
   ▼
[Find Speaker Actor (SpeakerTag)]
   │
   ▼
[Start Lip-Sync] ── Mesh: <Sprecher-Mesh>, Sound: Sound
```

### 3. Lip-Sync bei Voice Line Ended stoppen

`HandleVoiceEnd` schließt den Mund – setze die Kiefer-/Mund-Blendshape-Gewichte zurück auf null und stoppe den Treiber. Da `OnVoiceLineEnded` auch bei Skip und Abort feuert, deckt dieser eine Handler natürliches Ende, Spieler-Skip und Mitten-in-der-Zeile-Abbruch ab.

```text
[HandleVoiceEnd (SpeakerTag)]
   │
   ▼
[Find Speaker Actor (SpeakerTag)]
   │
   ▼
[Stop Lip-Sync] ── Mesh: <Sprecher-Mesh>
```

### 4. Am Dialog-Ende aufräumen

Binde auch `OnDialogueEnded`: die Voice-Events unbinden und jedes noch laufende Lip-Sync hart stoppen (falls eine Zeile abgeschnitten wurde). Das verhindert einen Mund, der nach Gesprächsende weiterklappt.

```text
[OnDialogueEnded]
   │
   ▼
[Stop Lip-Sync (alle Sprecher)]
[Voice-Events unbinden]
```

> 📸 **Bild-Platzhalter:** `lip-sync-bridge-ingame.png` — PIE-Screenshot: NPC spricht, Dialog-Widget zeigt die Zeile.
> *Setup:* PIE läuft. Viewport: NPC-Kopf mitten im Sprechen, Mund offen (Blendshape-getrieben). Dialog-Widget unten zeigt die aktuelle Zeile mit Typewriter-Animation. Caption-Hinweis: "Mund getrieben von OnVoiceLineStarted."

## Konkrete Hinweise je Lip-Sync-Typ

### Audio-Amplituden-getriebene Blendshapes

Der einfachste Ansatz: die RMS-Amplitude der spielenden Voice samplen und auf eine einzelne Kiefer-Öffnen-Blendshape mappen.

- Bei `OnVoiceLineStarted` einen Per-Tick-Sampler auf dem aktiven Audio des Sprechers starten (ein `USoundVisualizationStatics`-Envelope-Read oder deine eigene Amplituden-Sonde).
- Pro Tick das Kiefer-/Mund-offen-Morph-Target auf die normalisierte Amplitude setzen.
- Bei `OnVoiceLineEnded` den Sampler stoppen und das Morph zurück auf null lerpen.

Das funktioniert für jeden Voice-Asset-Typ – auch für den **Babel-Synth**-Platzhalter, da Amplituden-Sampling nicht unterscheidet, ob das Audio eine Aufnahme oder synthetisiert ist.

### Viseme-basierte Tools (OVR LipSync, Audio2Face, MetaHuman)

Diese Tools wollen das **Sound-Asset** (oder einen Live-Audio-Stream) vorab, um Viseme-/Phonem-Kurven zu berechnen.

- Bei `OnVoiceLineStarted` die `Sound`-Payload an den "Play and Analyze"-Entry-Point des Tools geben, damit es die Viseme-Kurve synchron zur Voice generiert, die MayDialogue bereits spielt.
- Die Viseme-Kurven des Sprechers aus dem Output des Tools treiben.
- Bei `OnVoiceLineEnded` die Wiedergabe/Analyse des Tools stoppen und die Viseme zurücksetzen.

{% hint style="warning" %}
**Doppelte Wiedergabe vermeiden.** Viseme-Tools spielen den Sound oft *auch* selbst ab. MayDialogue spielt die Voice der Zeile bereits. Nutze das Tool im "Analyze only"-/"Drive curves only"-Modus oder mute den eigenen Audio-Output des Tools, damit die Zeile nicht doppelt zu hören ist.
{% endhint %}

## Multiplayer-Hinweis (vor dem Koop-Release lesen)

Die Voice-Line-Events sind **server-only**, und es gibt **keinen Client-Mirror dafür**. `OnVoiceLineStarted` / `OnVoiceLineEnded` feuern auf der `UMayDialogueInstance`, die nur auf dem Server existiert — und in 1.0 spiegelt die `UMayDialogueParticipant` nur **Message**- und **Choice**-Daten an Clients, **nicht** den Voice-Line-Lifecycle. Es gibt keinen Voice-Line-Client-RPC, den man auf dem Client binden könnte.

Das heißt: Das obige Muster (`OnVoiceLineStarted` / `OnVoiceLineEnded` binden) funktioniert direkt in **Standalone** und auf einem **Listen-Server-Host** — wo der treibende Spieler auf dem Server ist — aber **nicht** auf einem Dedicated Server oder für einen Remote-Koop-Client. Auf einem Dedicated Server feuern diese Events, aber es gibt kein Audio zum Lip-Syncen; auf einem Remote-Client feuern sie gar nicht.

Um Lip-Sync auf einem Remote-Client zu treiben, nutze ein Signal, das den Client tatsächlich erreicht:

- **`OnMessageReceived`** *wird* an Clients gespiegelt (über den Client-RPC der Participant). Nutze es als „eine Zeile wird jetzt gesprochen"-Trigger und starte eine **Amplituden-Probe** gegen das lokal spielende Voice-Audio des Clients, um den Mund zu treiben — das braucht kein Server-Event und funktioniert auch für Babel-Platzhalter-Audio.
- Stoppe beim nächsten `OnMessageReceived` (die Zeile wechselte) und beim gespiegelten Dialog-Ende-Signal.

Kurz: Der Pfad über die `Sound`-Payload und die Voice-Line-Events ist eine **Single-Player-/Listen-Server-Host**-Bequemlichkeit. Für echtes Multiplayer-Lip-Sync treibe es stattdessen aus der gespiegelten Message + der lokalen Audio-Amplitude.

## Variation / Weiter gehen

- **Untertitel + Lip-Sync zusammen**: dieselbe `OnVoiceLineStarted`-Payload trägt den `Sound`; kombiniere sie mit `OnMessageReceived` (das den Text trägt) für synchrone Captions.
- **Per-Sprecher-Treiber**: auf `SpeakerTag` verzweigen, um verschiedene Charaktere an verschiedene Lip-Sync-Setups zu routen (ein MetaHuman-Held vs. ein einfacher Blendshape-NPC).
- **Emotions-bewusste Mundformen**: die `EmotionTags` der Zeile aus der Message-Payload lesen, um die Ruhe-Mundpose zu beeinflussen (ein Lächeln bei `Emotion.Happy`).

## Troubleshooting

**Mund bewegt sich nie.**
Die Voice-Events sind nicht gebunden, oder sie binden, nachdem die erste Zeile bereits gespielt hat. Binde bei `OnDialogueStarted` (das vor der ersten Zeile feuert). Bei einer reinen Text-Zeile mit ausgeschaltetem Babel spielt keine Voice, also feuern keine Voice-Events – nutze Amplituden-Sampling mit aktivem Babel oder treibe einen generischen Sprech-Loop stattdessen über `OnMessageReceived`.

**Mund klappt nach Dialog-Ende weiter.**
Du hast Lip-Sync nur bei `OnVoiceLineEnded` gestoppt, aber die letzte Zeile wurde übersprungen/abgebrochen, bevor das sauber feuerte. Stoppe immer zusätzlich bei `OnDialogueEnded` hart.

**Voice ist doppelt zu hören.**
Ein Viseme-Tool spielt den Sound zusätzlich zu MayDialogue. Schalte das Tool in den Analyze-only-/Curve-drive-only-Modus (siehe Warnung oben).

**Der Mund des falschen Charakters bewegt sich.**
Die `SpeakerTag → Actor`-Auflösung matcht den falschen Actor. Prüfe deine Sprecher-Registry oder nutze `GetActiveParticipantActors` und matche auf den Speaker-Tag der Participant.

## Siehe auch

- [VO-Profi-Hooks: Voice-Line-Events & MetaSound](../audio/vo-hooks.md) – die volle Event-Referenz, die diese Bindings nutzen.
- [NPC-Animation während Zeile](npc-animation-during-line.md) – Körper-Animation parallel zur Voice-Line.
- [Bridge & Lifecycle-Events](../runtime/bridge-events.md) – die breitere Schnittstelle für externe Integration.
