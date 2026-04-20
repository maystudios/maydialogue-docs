# Debug-Tipps

Wenn die Standard-Troubleshooting-Seite nicht reicht, hier die systematische Vorgehensweise.

## Output-Log filtern

UE-Output-Log öffnen (Window → Developer Tools → Output Log). Filter auf `LogMayDialogue`. Typische Meldungen:

| Meldung | Bedeutung |
| --- | --- |
| `StartDialogue: Asset X has no entry point` | Compile fehlgeschlagen oder Entry-Node fehlt. |
| `Cannot resolve participant with tag X` | Speaker-Tag matcht keinen Actor mit Participant-Komponente. |
| `Async node X timed out` | Wait-Node / PlayAnimation wartet zu lange. |
| `Variable X not declared in asset` | Setter auf nicht-deklariertem Namen. |
| `Choice index out of bounds` | Programmatisches SelectChoice mit ungültigem Index. |

## Breakpoint-getriebenes Debugging

1. Dialog-Asset öffnen.
2. Breakpoint auf den Node vor dem Problem setzen.
3. PIE starten, Dialog auslösen.
4. Bei Hit: Watch-Panel zeigt Variablen, Step Over zeigt nächsten Node.
5. Wiederhole, bis das Problem lokalisiert ist.

## Preview-Runner für Logik-Validierung

Wenn der Dialog im Preview-Runner läuft, aber in PIE nicht:

* Das Problem liegt **nicht** in der Dialog-Struktur.
* Es liegt in Participants, GAS-State, Widget-Setup oder Input.

Wenn der Dialog im Preview-Runner **auch** nicht läuft: das Problem ist strukturell.

## Variable-Watch

Im Debugger-Watch-Panel kannst du Variablen live sehen und ändern. Nützlich für:

* Abfragen, ob Choice-Requirements fehlschlagen wegen falschem Wert.
* Testen von Branch-Logik mit manuell gesetzten Werten.

## Outline für „wo ist der Fehler?"

Wenn der Validator einen Fehler meldet, aber du nicht findest wo: Outline-Panel aufmachen, nach Typ filtern (z.B. alle SayLines anzeigen), per Click-to-Jump durchgehen.

## Find-in-Dialogue

Wenn du einen bestimmten Text oder Tag suchst: **Ctrl+F** im Asset-Editor. Durchsucht Nodes, Choices, Kommentare.

## Console Commands

UE-Konsole (`~`-Taste):

```
log LogMayDialogue Verbose
```

Aktiviert ausführliche Dialog-Logs.

```
showdebug MayDialogue
```

(Wenn implementiert — Backlog: Debug-Overlay.)

## Performance

Wenn der Dialog „ruckelt":

* **Typewriter-Geschwindigkeit zu hoch?** `TypewriterCharsPerSecond` senken.
* **Portrait-Asset groß?** Prüfe Texture-Size in Asset-Details.
* **Babel-Sample-Quality**: zu viele Samples → Reduce-Priority.
* **Camera-Blend-Dauer**: kurze Dauer kann bei viel Blend-Parallelität stocken. Tweak.

## Replication-Debug (Multiplayer)

* **`Net Verbose LogMayDialogueNet`** für RPC-Tracing.
* **Net-Dormancy** auf dem Participant prüfen: Wenn der NPC „dormant" ist, kommen RPCs nicht an.

{% hint style="warning" %}
Multiplayer-Replication ist aktuell **Phase 1** (Core-Infra). Einzelne Pfade (insb. `ClientUpdateConversation`) sind als Backlog markiert. Erwarte Incomplete-Funktionalität in Multiplayer-Szenarien.
{% endhint %}

## Crash / Ensure-Hit

1. UE-Crash-Log kopieren (`Saved/Logs/*.log`).
2. Stack-Trace nach `UMayDialogueInstance`, `UMayDialogueNode_*`, `UMayDialogueParticipant` scannen.
3. Bei Null-Dereference: Check Instance-Lebensdauer (Weak-Ptrs).
4. Bei Array-Out-of-Bounds: Choice-Index-Validierung.

Meldet uns den Crash mit Log + Repro-Steps — siehe `BACKLOG.md`.

## Systematischer Isolations-Test

Wenn ein Dialog-Asset „kaputt" ist, **halbiert es**:

1. Lege eine Kopie an.
2. Lösche die zweite Hälfte der Nodes.
3. Compile & teste.
4. Fehler weg? Das Problem war in der zweiten Hälfte.
5. Wiederhole mit der verbleibenden Hälfte → Binär-Suche auf Node-Level.

## Fallback auf Slate-Widget

Wenn dein UMG-Widget das Problem ist:

1. **Project Settings → `bUseSlateDialogueWidget = true`** und `DefaultDialogueWidgetClass = None`.
2. PIE neu starten.
3. Läuft der Dialog im Slate-Widget?
4. Ja → Bug ist im UMG-Setup.
5. Nein → Bug ist tiefer (Runtime, Participant, Subsystem).
