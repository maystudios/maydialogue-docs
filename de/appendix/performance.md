---
description: Wie MayDialogue Frame-Zeit verbraucht, wie du sie misst und wo die Hitch-Risiken liegen.
---

# Performance

MayDialogue ist so gebaut, dass es im Leerlauf billig und unter Last vorhersehbar ist. Ein Dialog ist eine synchrone State-Machine, die nur dann Arbeit verrichtet, wenn etwas passiert — eine Node läuft, eine Choice wird ausgewertet, eine Stimme spielt. Zwischen diesen Momenten kostet er fast nichts.

Diese Seite zeigt dir, **wie du siehst**, wohin die Zeit geht (`stat MayDialogue`, Unreal Insights), erklärt das **asynchrone Pre-Warm**, das die wenigen wirklich teuren Operationen versteckt, und listet die **bekannten Hitch-Risiken** auf, damit du sie einplanen kannst.

{% hint style="info" %}
**Wer braucht diese Seite?** Die meisten Projekte müssen nie über Dialog-Performance nachdenken — ein einzelnes Gespräch ist im Frame-Budget ein Rundungsfehler. Lies sie, wenn du **viele Dialoge gleichzeitig** laufen lässt (Ambient-NPC-Geplauder quer durch eine Stadt), wenn du einen **Hitch siehst, sobald eine Zeile spielt**, oder wenn du vor dem Release einen Profiling-Durchlauf machst.
{% endhint %}

---

## Der No-Code-Weg: `stat MayDialogue`

Jedes nennenswerte Stück Dialog-Arbeit ist in einen benannten Counter gewickelt. So siehst du sie live:

1. Spiele im Editor (PIE) oder starte einen gepackten Build.
2. Öffne die Konsole (`~`) und tippe:

   ```text
   stat MayDialogue
   ```

3. Starte einen Dialog. Das Bildschirm-Overlay zeigt jetzt Pro-Frame-Zeiten für jeden Counter, mit Min/Max/Durchschnitt.

Zum Ausschalten tippst du `stat MayDialogue` erneut.

> 📸 **Bild-Platzhalter:** `stat-maydialogue-overlay.png` — Das `stat MayDialogue`-Overlay während eines Dialogs.
> *Setup:* PIE-Session mit aktiv laufendem Dialog (eine SayLine mit Babel-Stimme auf dem Bildschirm). Konsolenbefehl `stat MayDialogue` eingegeben. Das Stat-Overlay ist oben links sichtbar und zeigt die Zeilen `ContinueToNode`, `ExecuteNode` und `SayLine_*` mit Werten ungleich null. Frame-Zähler sichtbar. Eng auf das Overlay zuschneiden, damit die Zeilennamen lesbar sind.

---

## Die instrumentierten Counter

Alle Counter liegen in der **`MayDialogue`-Stat-Gruppe** (`STATGROUP_MayDialogue`), deklariert in `MayDialogueStats.h`. Jeder ist sowohl ein `stat`-Cycle-Counter (sichtbar über `stat MayDialogue`) als auch ein Unreal-Insights-CPU-Trace-Scope (sichtbar in Insights unter dem `MayDialogue::`-Namespace — siehe [Unreal Insights](#unreal-insights-tiefen-profiling) unten).

### Hot-Path-Counter (in der Quelle bestätigt)

| Counter (`stat`-Name) | Umschließt | Wann er feuert |
|---|---|---|
| **ContinueToNode** | `UMayDialogueInstance::ContinueToNode` | Die komplette Traversal-Schleife — jeder Schub von Node-zu-Node-Vorwärtsgang. Der oberste Dialog-Kostenpunkt. |
| **ExecuteNode** | Ein einzelner Node-Dispatch (`ExecuteNode`) in der Schleife | Einmal pro ausgeführter Node. |
| **RefreshPendingChoices** | `UMayDialogueInstance::RefreshPendingChoices` | Immer wenn gestagte Choices neu ausgewertet werden (z.B. eine Variable ändert sich, während auf eine Choice gewartet wird). |
| **SayLine_ResolveAudio** | `ResolveAudioSettings` — die vierstufige Audio-Kette | Einmal pro SayLine, die Audio spielt. |
| **SayLine_ResolveVoice** | `ResolveVoice` — Voice-Asset + Per-Culture-Lookup | Einmal pro SayLine. |
| **SayLine_BabelFallback** | `ExecuteBabelFallback` — Babel-Profil-Auflösung + Synth-Spawn | Nur wenn eine SayLine kein Voice-Asset hat und Babel aktiv ist. |
| **CameraFocus_Sequence** | `UMayDialogueNode_CameraFocus::ExecuteClientEffects` — Level-Sequence-Load + Spawn | Nur bei einer CameraFocus-Node im Sequence-Modus. **Hitch-Risiko** — siehe unten. |
| **Link_LoadTarget** | `UMayDialogueNode_Link::ExecuteNode` — synchroner Load des verlinkten Assets | Nur bei einer Link-Node, deren Ziel-Asset noch nicht im Speicher liegt. **Hitch-Risiko** — siehe unten. |
| **StartDialogue_PreWarm** | `UMayDialogueInstance::PreWarmAssets` — asynchrones Soft-Ref-Sammeln beim Start | Einmal pro `StartDialogue`. Billig — es *fordert* Loads nur an, es blockiert nicht. |

### Per-Frame-Tick- & Gauge-Counter

Die Counter oben sind **ereignisgetrieben** — sie feuern nur, wenn eine Node läuft. Die folgenden Counter messen **kontinuierliche Pro-Frame-Kosten** und die Live-Instanzanzahl, damit du ein Idle-Leak (ein Dialog, der ewig tickt) oder eine Concurrency-Spitze erkennst:

| Counter (`stat`-Name) | Umschließt | Liest sich als |
|---|---|---|
| **SubsystemTick** | `UMayDialogueSubsystem::Tick` | Pro-Frame-Kosten des Subsystem-Fan-outs über alle aktiven Instances. Null in Frames ohne aktiven Dialog (das Subsystem tickt nur, solange `ActiveDialogues.Num() > 0`). |
| **InstanceTick** | `UMayDialogueInstance::Tick` | Pro-Frame-Kosten des Ticks einer einzelnen Instance (Auto-Advance-Timer, Choice-Timeouts, Focus-Kamera-Tracking). |
| **BabelSynthTick** | `UMayDialogueBabelSynth::Tick` | Pro-Frame-Kosten einer aktiven Babel-Stimme (Continuous-Sync-Mode-Treiber). |
| **ActiveDialogues** (Gauge) | Die Länge des `ActiveDialogues`-Arrays am Subsystem | Eine *Anzahl*, keine Zeit — wie viele Dialoge in diesem Frame gleichzeitig laufen. |

{% hint style="info" %}
Die vier Tick-/Gauge-Counter in der Tabelle direkt darüber (`SubsystemTick`, `InstanceTick`, `BabelSynthTick` und das `ActiveDialogues`-Gauge) sind **jetzt schon in der Quelle vorhanden** — sie sind in `MayDialogueStats.h` neben den Hot-Path-Countern deklariert. Die Funktionen, die sie umschließen (`UMayDialogueSubsystem::Tick`, `UMayDialogueInstance::Tick`, `UMayDialogueBabelSynth::Tick`), ticken heute. Nur die repräsentativen *Messwerte* in der Tabelle am Ende dieser Seite stehen noch unter Vorbehalt des 1.0-Perf-Durchlaufs.
{% endhint %}

---

## Unreal Insights (Tiefen-Profiling)

`stat MayDialogue` gibt dir Pro-Frame-Durchschnitte auf dem Bildschirm. Für eine Frame-für-Frame-Timeline — genau welche Node welche Mikrosekunden gekostet hat und wie sie sich mit dem Rest deines Spiels überlappt — nimmst du **Unreal Insights**.

Jeder MayDialogue-Counter ist auch ein CPU-Trace-Scope im `MayDialogue::`-Namespace (z.B. `MayDialogue::ContinueToNode`, `MayDialogue::ExecuteNode`, `MayDialogue::StartDialogue_PreWarm`). So fängst du sie ein:

1. Starte dein Spiel mit aktiviertem CPU-Tracing:

   ```text
   -trace=cpu,frame,bookmark
   ```

2. Reproduziere die Dialog-Szene, die du profilen willst.
3. Öffne die resultierende `.utrace` in Unreal Insights und filtere die Timing-Tracks nach `MayDialogue`.

> 📸 **Bild-Platzhalter:** `insights-maydialogue-trace.png` — Unreal-Insights-Timeline gefiltert auf MayDialogue-Scopes.
> *Setup:* Unreal Insights geöffnet auf einer mit `-trace=cpu,frame` aufgenommenen `.utrace`. Die Timing-Insights-Ansicht ist (im Suchfeld) auf `MayDialogue` gefiltert. Sichtbare Scopes: `MayDialogue::ContinueToNode` mit verschachtelten `MayDialogue::ExecuteNode`-Blöcken und einem `MayDialogue::SayLine_ResolveVoice`-Kind. Einen Frame wählen, in dem eine SayLine gefeuert hat, damit die Verschachtelung klar ist.

---

## Wie Pre-Warm die teuren Teile versteckt

Drei Operationen in einem Dialog sind wirklich teuer, weil sie die Platte anfassen: das Laden eines **Attenuation-/Sound-Class-/Babel-Profil**-Assets für Audio, das Laden einer **Level Sequence** für einen CameraFocus und das Laden eines **verlinkten Dialog-Assets** für eine Link-Node. Synchron im Moment der Nutzung ausgeführt, kann jede davon einen Frame-Hitch verursachen.

Um diese Kosten zu verstecken, ruft `UMayDialogueInstance::StartDialogue` **`PreWarmAssets`** auf, bevor je eine Node läuft. Es geht jede kompilierte Node im Asset durch, sammelt die per Soft-Ref referenzierten Assets jeder Node und feuert einen **einzelnen Fire-and-Forget-Async-Load** über den StreamableManager:

```text
StartDialogue
    └─> PreWarmAssets(Asset)            // STAT: StartDialogue_PreWarm
          ├─ Plugin-Default-Soft-Refs sammeln (Attenuation, Sound Class, Babel-Profil)
          ├─ CompiledNodes durchgehen:
          │     SayLine    → Speaker-Attenuation / -Sound-Class / -Babel-Profil
          │     CameraFocus → Level Sequence
          │     Link        → Ziel-Dialog-Asset
          └─ StreamableManager::RequestAsyncLoad(alle Pfade)   // asynchron, nicht blockierend
```

Bis der Spieler eine SayLine-, CameraFocus- oder Link-Node erreicht — meist mehrere Sekunden Gespräch später — sind die Assets bereits im Speicher. Der Per-Node-Code ruft weiterhin `LoadSynchronous` als **Korrektheits-Fallback**, findet das Asset aber im Cache und kehrt sofort zurück, statt die Platte anzufassen.

{% hint style="info" %}
Pre-Warm ist **Best-Effort**, keine Garantie. Wenn ein Spieler schneller durch einen Dialog rast, als der Async-Load fertig wird, oder wenn der Asset-Manager nicht verfügbar ist (Headless-Test-Umgebungen), läuft trotzdem der synchrone Fallback und kann hitchen. Für sehr große Level Sequences oder tief verkettete Link-Ziele siehe die Gotchas unten.
{% endhint %}

---

## Bekannte Hitch-Risiken

| Risiko | Warum | Gegenmaßnahme |
|---|---|---|
| **CameraFocus im Sequence-Modus** | Laden + Spawnen einer Level Sequence in dem Frame, in dem die Node läuft (`CameraFocus_Sequence`). Große Sequences mit vielen Tracks sind der schlimmste Fall. | Pre-Warm sammelt die `CameraSequence`-Soft-Ref bei `StartDialogue`. Halte Sequences schlank; setze den CameraFocus ein paar Zeilen ins Gespräch hinein, damit Pre-Warm Zeit hat. |
| **Link-Ziel-Load** | Eine Link-Node, deren `TargetDialogueAsset` noch nicht im Speicher liegt, lädt es synchron (`Link_LoadTarget`). | Pre-Warm sammelt das unmittelbare Link-Ziel. Achtung: Pre-Warm geht nur die kompilierten Nodes **eines** Assets durch — eine Kette von Links-in-Links-in-Links wärmt nur den ersten Sprung vor. |
| **Babel-Synth-Spawn** | Der erste Frame, in dem eine Babel-Stimme startet (`SayLine_BabelFallback`), alloziert und konfiguriert den Synth. | Billig im Vergleich zu einem echten Voice-Asset, aber pro Zeile. Bei dichtem Ambient-Babel-Geplauder beobachte `BabelSynthTick` × Anzahl gleichzeitiger Stimmen. |
| **Viele gleichzeitige Dialoge** | Jeder aktive Dialog tickt (`SubsystemTick` fächert zu `InstanceTick` auf). | Die Kosten skalieren linear mit `ActiveDialogues`. Beobachte das `ActiveDialogues`-Gauge; deckle Ambient-Gespräche, wenn du Dutzende spawnst. |

---

## Eine Dialog-Szene profilen — ein Rezept

1. **Reproduziere den Worst Case.** Baue die Szene mit den meisten gleichzeitigen Dialogen / dem schwersten CameraFocus auf, den du auslieferst.
2. **Beobachte `stat MayDialogue`** live. Notiere, welcher Counter dominiert. Eine Spitze bei `CameraFocus_Sequence` oder `Link_LoadTarget` deutet auf einen Load, den Pre-Warm nicht abgedeckt hat (zu schnell, oder eine tiefe Link-Kette).
3. **Bestätige mit `stat Unit`**, dass die Spitze tatsächlich in der `Game`-Thread-Zeit auftaucht — Dialog läuft auf dem Game-Thread.
4. **Nimm einen Insights-Trace auf** (`-trace=cpu,frame`) für eine frame-genaue Sicht auf die Verschachtelung und um zu sehen, was *sonst* im Hitch-Frame lief.
5. **Prüf das Gauge.** Ist `ActiveDialogues` höher als erwartet, hast du Dialoge, die nie endeten — such nach einer Wait-Node ohne Resume oder einer Instance, deren Participant ohne `EndPlay` zerstört wurde.

---

## Messwerte

Die Struktur liefert jetzt; die Zahlen kommen mit dem 1.0-Performance-Durchlauf auf repräsentativer Zielhardware.

> Messwerte werden im 1.0-Perf-Durchlauf gefüllt

| Was | Konfiguration | Kosten | Notizen |
|---|---|---|---|
| Subsystem-Tick — Idle | Kein aktiver Dialog | _TBD_ | Sollte 0 sein — das Subsystem tickt nicht, solange `ActiveDialogues.Num() == 0`. |
| Subsystem-Tick — aktiv | 1 aktiver Dialog, keine Async-Arbeit in diesem Frame | _TBD_ | `SubsystemTick` + ein `InstanceTick`. |
| Instance-Tick — aktiv | 1 Instance, Auto-Advance/Focus-Tracking läuft | _TBD_ | `InstanceTick`. |
| Babel-Synth — pro aktiver Stimme | 1 Babel-Stimme im Continuous-Modus | _TBD_ | `BabelSynthTick`, CPU pro Stimme. |
| Speicher — pro aktivem Dialog | 1 `UMayDialogueInstance` + Async-State | _TBD_ | Instance-Objekt + Per-Node-AsyncState-Einträge. |
| Widget-Konstruktionskosten | Erster UMG-Widget-Auto-Spawn bei `StartDialogue` | _TBD_ | Einmalig pro Subsystem-Lebensdauer (danach wiederverwendet). |
| StartDialogue (kalt) | Erster Start eines Assets, Assets nicht resident | _TBD_ | Dominiert von `StartDialogue_PreWarm`-Request + Ausführung der ersten Node. |
| StartDialogue (warm) | Asset bereits resident | _TBD_ | Pre-Warm findet alles im Cache. |
