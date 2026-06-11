---
description: Voice-Line-Lifecycle-Events und MetaSound-Parameter-Binding – die Hooks, die ein VO-/Audio-Profi braucht, um Lip-Sync, Mixing und prozedurale Stimmen zu treiben.
---

# VO-Profi-Hooks: Voice-Line-Events & MetaSound

## Wann brauche ich das?

Wenn du Dialog-Voice in ein größeres Audio- oder Animations-System einbindest und präzise, frame-genaue Hooks brauchst:

- Ein **Lip-Sync**- oder Facial-Animation-System muss genau wissen, wann eine Voice-Line startet und stoppt.
- Ein **Audio-Mixer** will Musik ducken oder einen "Charakter spricht"-Snapshot für die Dauer jeder Zeile triggern.
- Ein **MetaSound**-Voice-Asset soll auf Sprecher, Emotion und Pitch der Zeile reagieren – z.B. eine Kreaturenstimme, die ihr Timbre je nach Emotion ändert.

MayDialogue stellt dafür zwei Dinge bereit: **Voice-Line-Lifecycle-Delegates** auf der Dialog-Instanz und **automatisches MetaSound-Parameter-Binding**, wenn das Voice-Asset eine MetaSound Source ist.

## Voice-Line-Lifecycle-Events

Um jede Voice-Line feuern zwei Events, parallel zu den bestehenden Message-Events:

| Event | Feuert wenn |
|---|---|
| `OnVoiceLineStarted` | Die Voice-Wiedergabe einer Zeile beginnt (nachdem die Message präsentiert wurde, wenn das Audio tatsächlich startet). |
| `OnVoiceLineEnded` | Die Voice der Zeile endet – natürlich oder weil die Zeile übersprungen/abgebrochen wurde. |

Jedes gibt es in einer **dynamischen** Form (für UMG / Blueprint) und einer **nativen** Form (für C++ / Slate). Beide tragen dieselbe Payload: die Node-GUID, den Speaker-Tag und das Sound-Asset.

```text
StartDialogue
   → OnMessageReceived            (Text landet im UI)
   → OnVoiceLineStarted           ← Voice-Audio beginnt   ◀── hier Lip-Sync AN
        … Zeile spielt …
   → OnVoiceLineEnded             ← Voice-Audio endet      ◀── hier Lip-Sync AUS
   → (advance)
```

> 📸 **Bild-Platzhalter:** `vo-hooks-event-timeline.png` — Timeline-Grafik der Events einer Zeile.
> *Setup:* Eine Grafik erstellen (kein Editor-Screenshot). Ein horizontaler Zeit-Pfeil. Marker in Reihenfolge: `OnMessageReceived`, `OnVoiceLineStarted` (grün), ein schattiertes "Voice spielt"-Band, `OnVoiceLineEnded` (rot), `advance`. Die grünen/roten Marker mit "Lip-Sync AN / AUS" beschriften. Weißer Hintergrund.

### Warum "AfterVoice" jetzt robust ist

Der `AfterVoice`-Advance-Mode (eine SayLine, die weitergeht, wenn ihre Voice fertig ist) wird jetzt von derselben Event-basierten Voice-Ende-Erkennung getrieben wie `OnVoiceLineEnded`, statt von einer Polling-Schätzung. Das Ergebnis: Eine Zeile mit `AfterVoice` geht genau dann weiter, wenn die Audio-Komponente den Abschluss meldet – zuverlässig auch bei variabel langen lokalisierten Takes.

Der Horror-Sample-Dialog (`DA_Horror_Corridor`) zeigt das durchgängig: Die Zähl-Zeile des Attendant trägt einen echten VO-Take (`A_Attendant_Count`, unter `/MayDialogue/Samples/Audio/`) und geht per `AfterVoice` weiter, sodass die nächste Zeile wartet, bis die aufgenommene Stimme fertig ist.

## Zero-Code-Weg (Blueprint)

Du kannst die Events ohne eine Zeile C++ konsumieren.

1. Hol dir die aktive `UMayDialogueInstance` (z.B. über `UMayDialogueSubsystem → Get Active Dialogue` oder fang sie aus dem `OnDialogueStarted`-Event ab).
2. **Bind Event to On Voice Line Started** und **On Voice Line Ended**.
3. Im gebundenen Event die Payload (`NodeGuid`, `SpeakerTag`, `Sound`) lesen und dein System treiben – eine Lip-Sync-Komponente starten, einen Mixer-State setzen, einen Untertitel spawnen usw.

> 📸 **Bild-Platzhalter:** `vo-hooks-blueprint-bind.png` — Blueprint-Graph, der beide Voice-Line-Events an der Instanz bindet.
> *Setup:* Blueprint-Event-Graph. `OnDialogueStarted` → `Bind Event to OnVoiceLineStarted` und `Bind Event to OnVoiceLineEnded` an der Instanz. Zwei Custom-Events darunter: "HandleVoiceStart" (zieht SpeakerTag + Sound) und "HandleVoiceEnd". Die roten Event-Dispatcher-Pins mit den Custom-Events verdrahtet zeigen.

{% hint style="warning" %}
**Multiplayer-Vorbehalt — diese Events sind server-only.** `OnVoiceLineStarted` / `OnVoiceLineEnded` feuern auf der `UMayDialogueInstance`, die nur auf dem **Server** lebt (die Voice-Wiedergabe und die Audio-Komponente liegen ebenfalls dort). Es gibt in 1.0 **keinen** Voice-Line-Client-RPC und **kein** gespiegeltes Voice-Line-Event auf `UMayDialogueParticipant` — nur Message- und Choice-Daten werden an Clients gespiegelt. Auf einem **Dedicated Server** feuern diese Events also, aber es gibt kein lokales Audio, und auf einem **Remote-Client** feuern sie gar nicht.

Für eine client-seitige Reaktion (z.B. Lip-Sync auf einem remote-besessenen Charakter) treibe sie über ein Signal, das den Client *tatsächlich* erreicht — das gespiegelte `OnMessageReceived` (das den Text und die Emotion-Tags der Zeile trägt) — oder lass eine Amplituden-Probe gegen das eigene spielende Audio des Clients laufen. Das ehrliche MP-Muster zeigt das [Lip-Sync-Rezept](../recipes/lip-sync-bridge.md).
{% endhint %}

## API-Referenz — Voice-Line-Delegates

Deklariert auf `UMayDialogueInstance` (Modul `MayDialogue`):

| Member | Form | Signatur (Params) |
|---|---|---|
| `OnVoiceLineStarted` | Dynamic (`BlueprintAssignable`) | `(FGuid NodeGuid, FGameplayTag SpeakerTag, USoundBase* Sound)` |
| `OnVoiceLineEnded` | Dynamic (`BlueprintAssignable`) | `(FGuid NodeGuid, FGameplayTag SpeakerTag, USoundBase* Sound)` |
| `OnVoiceLineStartedNative` | Native (C++ / Slate) | `(UMayDialogueInstance*, FGuid, FGameplayTag, USoundBase*)` |
| `OnVoiceLineEndedNative` | Native (C++ / Slate) | `(UMayDialogueInstance*, FGuid, FGameplayTag, USoundBase*)` |

Die nativen Varianten tragen die Sender-Instanz als ersten Parameter (gleiche Konvention wie `OnNodeReachedNative` / `OnEventFiredNative`), sodass ein einzelner Subscriber gleichzeitig laufende Dialoge unterscheiden kann.

```cpp
// C++: das native Voice-Start-Event binden.
Instance->OnVoiceLineStartedNative.AddWeakLambda(this,
    [this](UMayDialogueInstance* Sender, FGuid NodeGuid,
           FGameplayTag SpeakerTag, USoundBase* Sound)
    {
        StartLipSyncFor(SpeakerTag, Sound);
    });
```

## MetaSound-Parameter-Binding

Wenn das aufgelöste Voice-Asset einer Zeile eine **MetaSound Source** ist, schiebt das Plugin pro Zeile Parameter in den MetaSound bei der Wiedergabe – über das Parameter-Interface der Audio-Komponente (`ISoundParameterControllerInterface`). Dein MetaSound-Graph kann sie als Input-Parameter lesen.

| Parameter-Name | Typ | Wert |
|---|---|---|
| `MayDialogue.SpeakerTag` | String | Der `SpeakerTag` der sprechenden Node als voller hierarchischer Tag-Name (z.B. `Dialogue.Speaker.Guard`); leer, wenn nicht gesetzt. |
| `MayDialogue.Emotion` | String | Der **erste** gesetzte Emotion-Tag (z.B. `Emotion.Angry`); leer, wenn keiner. |
| `MayDialogue.EmotionTags` | String | **Alle** Emotion-Tags, komma-getrennt verbunden (z.B. `Emotion.Angry,Emotion.Loud`); leer, wenn keine. |
| `MayDialogue.Pitch` | Float | Der aufgelöste Pitch-Multiplikator der Zeile (Plugin × Speaker × Participant × Node). |

Alle vier Parameter-Namen sind mit `MayDialogue.` **präfigiert**, und die Tag-Werte werden als **String**-Inputs geschoben (das Parameter-Interface bietet keinen Name-Setter – ein MetaSound-Graph kann selbst String→Name konvertieren, wenn er einen Name-Pin braucht). Dieses Binding ist automatisch – du rufst nichts auf. Erstelle eine MetaSound Source mit passenden Input-Parametern, weise sie als Voice der Zeile (oder des Sprechers) zu, und die Werte kommen bei jedem Abspielen der Zeile an.

### Schritt-für-Schritt — Ein Pitch-/Gain-reaktiver MetaSound

1. **MetaSound Source erstellen** (`MS_CreatureVoice`). Rechtsklick im Content-Browser → Sounds → MetaSound Source.
2. **Input-Parameter hinzufügen**, exakt passend zu den Namen oben (inklusive `MayDialogue.`-Präfix):
   - `MayDialogue.Pitch` (Float, Default `1.0`)
   - `MayDialogue.SpeakerTag` (String) – optional, für Branching
   - `MayDialogue.Emotion` (String) – optional, der erste Emotion-Tag
   - `MayDialogue.EmotionTags` (String) – optional, alle Emotion-Tags komma-getrennt
3. **`MayDialogue.Pitch` verdrahten** in den Pitch-Shift deines Generators (oder in ein `Multiply` am Pitch-Input des Wave-Players). Jetzt treibt der aufgelöste Pitch-Multiplikator der Zeile den Synth.
4. (Optional) **`MayDialogue.Emotion` nutzen**, um zwischen zwei Timbres zu wählen: ein `String Equals "Emotion.Angry"` → Gain-Crossfade zwischen einer harschen und einer weichen Layer.
5. **`MS_CreatureVoice` zuweisen** als `DialogueVoice` an der SayLine (oder als Default-Voice des Sprechers).
6. **Preview** der Zeile im [Preview-Runner](../editor/preview-runner.md): Der Synth reagiert auf den Per-Zeile-Pitch und die Emotion.

> 📸 **Bild-Platzhalter:** `vo-hooks-metasound-graph.png` — Ein MetaSound-Source-Graph, der die Input-Parameter `Pitch` und `EmotionTags` liest.
> *Setup:* MetaSound-Editor offen auf `MS_CreatureVoice`. Sichtbar: Input-Parameter `Pitch (Float)`, `EmotionTags (String)`, `SpeakerTag (String)` im Inputs-Panel. Der `Pitch`-Input in einen Pitch-Shift-/Multiply-Node verdrahtet, der den Output speist. Roter Pfeil auf den `Pitch`-Input-Pin. Sauberer Graph, eine Handvoll Nodes.

> 📸 **Bild-Platzhalter:** `vo-hooks-sayline-metasound-assigned.png` — SayLine-Details mit einer MetaSound Source als Voice zugewiesen.
> *Setup:* SayLine-Node im Dialog-Editor ausgewählt. Details-Panel-Bereich "SayLine → Audio": `DialogueVoice`-Slot hält `MS_CreatureVoice` (MetaSound-Icon). `PitchMultiplier = 1.3`, `EmotionTags = Emotion.Angry`. Roter Pfeil auf den Voice-Slot.

## Edge-Cases & Stolpersteine

**Parameter erreichen nur MetaSound Sources.** Ist das Voice-Asset eine einfache `SoundWave` oder ein `SoundCue`, ist der Parameter-Push ein No-Op (diese Formate haben keine Parameter-Inputs). Der `PitchMultiplier` gilt trotzdem über den normalen Audio-Pfad – nur die *benannten* Parameter brauchen einen MetaSound-Graph, der sie deklariert.

**Parameter-Namen müssen exakt passen, Präfix inklusive.** Ein MetaSound-Input mit Namen `Pitch` (ohne `MayDialogue.`-Präfix) oder `maydialogue.pitch` (falsche Schreibweise) erhält den Wert nicht. Die vollen Namen `MayDialogue.Pitch` / `MayDialogue.SpeakerTag` / `MayDialogue.Emotion` / `MayDialogue.EmotionTags` aus der Tabelle oben einhalten.

**`OnVoiceLineEnded` feuert bei Skip und Abort, nicht nur bei natürlichem Ende.** Wenn der Spieler die Zeile überspringt oder der Dialog mitten in der Zeile abbricht, feuert das Ende-Event trotzdem, sodass dein Lip-Sync sauber abschaltet. Behandle es als "die Voice dieser Node spielt nicht mehr", nicht als "das Audio hat sein letztes Sample erreicht".

**Babel schiebt keine MetaSound-Parameter.** Wenn eine Zeile kein Voice-Asset hat und der Babel-Synth einspringt, feuern die Voice-Line-*Events* trotzdem (Lip-Sync sieht also eine "Voice"), aber es gibt kein MetaSound-Parameter-Binding – Babel ist sein eigener Synthese-Pfad. Siehe [Babel-System](babel-system.md).

**Kein Voice-Asset, Babel aus → keine Voice-Events.** Eine reine Text-Zeile mit ausgeschaltetem `bEnableBabelVoice` spielt kein Audio, also feuern `OnVoiceLineStarted` / `OnVoiceLineEnded` dafür nicht. Treibe rein visuelle Reaktionen stattdessen über `OnMessageReceived` / `OnNodeReached`.

## Siehe auch

- [Lip-Sync über die Bridge](../recipes/lip-sync-bridge.md) – das konkrete Rezept, das diese Events konsumiert.
- [Node-Overrides](node-overrides.md) – woher der Per-Zeile-Pitch-Multiplikator kommt.
- [Babel-System](babel-system.md) – der prozedurale Fallback, wenn kein Voice-Asset gesetzt ist.
