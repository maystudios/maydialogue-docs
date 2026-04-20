# Häufige Probleme

## Dialog startet nicht {#dialog-start}

### Symptome

* `StartDialogue` wird aufgerufen, aber nichts passiert.
* Log-Warning: *„Dialogue asset has no entry point"* oder *„Cannot resolve participants"*.

### Prüfe

1. **Asset compiled?** Drücke **Compile** im Asset-Editor. Validator-Errors beheben.
2. **Entry-Node vorhanden?** Jedes Asset braucht genau einen.
3. **Participant-Komponenten gesetzt?**
   * Spieler-Pawn hat `UMayDialogueParticipant` mit `ParticipantTag = Dialogue.Speaker.Player`.
   * NPC hat `UMayDialogueParticipant` mit dem korrekten Tag.
4. **Tag-Matching**: SayLine-Speaker-Tags müssen zu existierenden Participant-Tags passen.
5. **Ein anderer Dialog läuft bereits?** `Subsystem->IsAnyDialogueActive()` abfragen.

## Widget erscheint nicht {#widget}

### Symptome

* Dialog läuft laut Log, aber keine UI sichtbar.

### Prüfe

1. **Slate-Widget abgeschaltet, aber UMG nicht gesetzt?**
   * Project Settings → MayDialogue → `bUseSlateDialogueWidget` oder `DefaultDialogueWidgetClass`.
2. **UMG-Widget hat fehlende BindWidget-Slots?** Öffne dein Widget im Designer, prüfe die Namen (`DialogFrameWidget`, `TextWidget`, …).
3. **Widget wird nach Level-Travel nicht neu gebunden** (bekannter Bug). Workaround: `StopAllDialogues` beim Level-Wechsel aufrufen.

## Choices fehlen {#choices}

### Symptome

* PlayerChoice-Node aktiv, aber keine Buttons erscheinen.

### Prüfe

1. **Alle Choices `FailedAndHidden`?** Keine der Choices hat Requirements, die passen. Prüfe im Preview-Runner mit simulierten Tags.
2. **ChoiceButtonClass leer?** Im `UMayDialogueWidget_ChoiceList`-Widget (Blueprint): `ChoiceButtonClass`-Property gesetzt?
3. **`ChoiceListWidget` nicht gebunden?** Top-Level-Widget hat kein ChoiceListWidget-BindSlot → Fallback auf Legacy-Monolith-Pfad.

## Audio läuft nicht {#audio}

### Symptome

* SayLine erscheint als Text, aber keine Voice.

### Prüfe

1. **Voice-Asset für aktuelle Culture hinterlegt?** `DialogueVoice[CultureKey]` prüfen.
2. **AudioComponent gespawnt?** Der Speaker-Actor muss einen `USoundAttenuation` akzeptieren (nicht hinter Wand-Occlusion begraben).
3. **SoundClass-Override ist still?** Der `SoundClass`-Mixer kann gemutet sein.
4. **`bForce2D`-Mischung?** 2D-Sound respektiert kein 3D-Voulme. Check globales Volume-Slider.

## Babel läuft nicht

### Symptome

* SayLine ohne Voice-Asset, aber kein Babel hörbar.

### Prüfe

1. **`bEnableBabelVoice = true`** in Project Settings.
2. **BabelProfile gesetzt?** Am Speaker oder als `DefaultBabelProfile` in Settings.
3. **BlipSounds leer UND `bUseProceduralDefaults=false`**: kein Sound. Entweder Samples zuweisen oder Procedural aktivieren.
4. **Component-Pfad: `OnCharacterRevealed` nicht manuell verdrahtet.** Im Blueprint das Event am `TextWidget` an `BabelSynth->OnCharacterRevealed` binden.

## Variable wird nicht übernommen {#variables}

### Symptome

* SetVariable-Node läuft, aber Requirement-Check liefert alte Werte.

### Prüfe

1. **Scope korrekt?** Dialogue-Scope-Variablen überleben Dialog-Ende NICHT. Wenn du sie über Dialoge hinaus brauchst: Participant-Scope.
2. **Backlog-Item 8**: SetVariable-Node schreibt aktuell nur Dialogue-Scope. Workaround: Blueprint-SideEffect nutzen, der `Part->SetPersistentXxx` aufruft.
3. **Variable deklariert?** Im Variables-Panel anlegen, bevor ein Node sie referenziert.
4. **Typ-Mismatch?** Validator warnt bei inkonsistenten Typen.

## Dialog bleibt hängen

### Symptome

* Dialog beginnt, aber nach einer bestimmten Zeile passiert nichts mehr.

### Prüfe

1. **Async-Node wartet auf Event, das nicht kommt?** Wait-Node mit `WaitEventTag`, aber das Event wird nie gefeuert.
2. **PlayAnimation mit `bWaitForMontageEnd=true`, aber Montage existiert nicht?**
3. **Advance-Mode `AfterVoice`, aber kein Voice-Asset?** Dialog wartet auf Voice-End-Callback, der nie kommt.

Im Debugger: Breakpoint auf den Node davor setzen, Step Over beobachten, welcher Node nicht advanced.

## Compile-Fehler ohne klaren Grund

### Symptome

* Compiler-Results-Tab zeigt Error, aber der Node-Click springt auf den falschen Node.

### Prüfe

1. **Alle Assets gespeichert?** Unsaved-Changes in anderen Dialog-Assets verwirren manchmal Cross-References.
2. **Engine-Restart** nach großen Änderungen (insbesondere an Plugin-Code).

## Sub-Graph navigiert nicht zurück

### Symptome

* SubGraph wird betreten, Exit erreicht, aber Dialog endet statt zum Caller zurückzukehren.

### Prüfe

1. **`bReturnAfterExit = true`?** Standard sollte `true` sein.
2. **SubGraph-Entry existiert?** Validator warnt, wenn nicht.

## „Other May Dialogue Participant is null"

### Symptome

* `StartDefaultDialogue(Other)` scheitert, Log-Warning über leeren Other-Param.

### Prüfe

1. Hat der `Other`-Actor eine Participant-Komponente?
2. Wurde sie zur Laufzeit bereits angelegt (bei dynamisch gespawnten Actors)?

Lösung: `AddComponentByClass<UMayDialogueParticipant>` in BeginPlay des Actors, oder Participant als Default-Component im Blueprint.
