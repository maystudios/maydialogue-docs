---
description: Alle wichtigen MayDialogue-Begriffe alphabetisch erklärt.
---

# Glossar

Dieses Verzeichnis definiert die Fachbegriffe, die in der Dokumentation und im Editor verwendet werden. Wenn du einen Begriff im Graph oder in einem Fehler siehst und nicht weißt, was er bedeutet, schau hier nach.

---

### Advance Mode

Legt fest, wie der Dialog vom aktuellen zum nächsten Node wechselt. Verfügbare Modi:

| Modus | Verhalten |
| --- | --- |
| `Manual` | Spieler-Input (z. B. Taste drücken) |
| `Timer` | Automatisch nach einer konfigurierten Zeit |
| `AfterVoice` | Wartet, bis das Voice-Asset vollständig abgespielt wurde |
| `AfterAnimation` | Wartet auf das Ende einer Montage |
| `Immediate` | Wechselt sofort ohne Pause |

Jeder SayLine-Node kann den globalen Advance Mode pro Node überschreiben (`AdvanceModeOverride`).

---

### Asset

Ein `UMayDialogueAsset` — das Blueprint-Asset, das du im Content-Browser erstellst und im Dialog-Editor öffnest. Es enthält den vollständigen Graph (Nodes, Links), alle Speaker-Definitionen und die Variable-Deklarationen des Dialogs.

---

### Bridge

`IMayDialogueBridge` — ein Interface, das externe Systeme (z. B. Quest-System, Inventory, eigene Logik) mit einem laufenden Dialog verbinden kann. Über die Bridge lassen sich Choices programmatisch auswählen, Events feuern und Variablen lesen. Nützlich, wenn du ein Dialog-System in einen größeren Game-Loop integrieren willst.

---

### Choice

Eine einzelne Antwort-Option in einem PlayerChoice-Node. Jede Choice kann Requirements, Tags und einen eigenen Output-Pin haben. Choices werden als Sub-Nodes im PlayerChoice-Node angezeigt.

Siehe auch: [FailedAndHidden](#failedandhidden), [FailedButVisible](#failedbut-visible), [Player Choice](#player-choice).

---

### Cue

Ein kurzes Ereignis, das während eines Dialogs ausgelöst wird — z. B. ein Gameplay-Tag-Event, ein Sound, eine Kamera-Shake. Cues sind nicht mit dem GAS-Gameplay-Cue-System zu verwechseln; im MayDialogue-Kontext ist ein Cue ein generisches Signal an externe Systeme.

---

### Dialogue-Scope

Variablen, die nur während der laufenden Dialog-Instance existieren. Wird der Dialog beendet (oder abgebrochen), werden Dialogue-Scope-Variablen verworfen. Für persistente Werte: [Participant-Scope](#participant-scope) nutzen.

---

### Emotion-Tag

Ein `FGameplayTagContainer`-Feld auf SayLine-Nodes. Emotion-Tags sind hierarchische Meta-Tags, die beschreiben, wie eine Zeile gespielt wird — z. B. `Dialogue.Emotion.Angry`, `Dialogue.Emotion.Scared`. Das Widget, das Audio-System und Animationen können Emotion-Tags auswerten, um das Erscheinungsbild der Zeile anzupassen.

---

### Entry

Der Start-Node eines Dialog-Assets. Jedes Asset hat genau einen Entry-Node. Er hat keinen Input-Pin und einen Output-Pin, der auf den ersten inhaltlichen Node zeigt. Fehlt der Entry-Node, schlägt der Compile fehl.

---

### Exit

Der End-Node eines Dialog-Assets oder Sub-Graphen. Er hat einen Input-Pin und keinen Output-Pin. Wenn der Dialog-Flow den Exit erreicht, wird die Instance beendet (oder bei Sub-Graphs: zum rufenden Graph zurückgekehrt). Exit-Nodes können einen Completion-Status tragen (z. B. `Completed`, `Failed`).

---

### FailedAndHidden

Ein Requirement-Ergebnis: Die Bedingung ist nicht erfüllt, und die Choice oder der Node ist für den Spieler komplett unsichtbar. Wird kein anderer Pfad angeboten, kann ein PlayerChoice-Node leer erscheinen.

---

### FailedButVisible

Ein Requirement-Ergebnis: Die Bedingung ist nicht erfüllt, aber die Choice ist im Widget sichtbar — gesperrt, mit visuellem Feedback (z. B. grauer Button, Schloss-Icon). Der Spieler sieht die Option, kann sie aber nicht auswählen.

---

### Instance

Ein `UMayDialogueInstance` — das laufende Objekt eines Dialogs. Wird beim Start erzeugt, verwaltet den aktuellen Node-Pointer, Variablen und den Scope-Stack. Wird beim Ende (Exit-Node) oder Abort zerstört. Pro laufendem Dialog gibt es genau eine Instance.

---

### Link

Ein Node-Typ, der den Dialog-Flow in ein anderes Dialog-Asset springt. Links ermöglichen es, Dialoge modular zu strukturieren und zwischen Assets zu navigieren. Optional kehrt der Flow nach dem Ende des Ziel-Assets zum rufenden Asset zurück (`bReturnAfterExit`).

---

### Node

Ein einzelnes Element im Dialog-Graph. Nodes sind die Bausteine eines Dialogs — von SayLines über Choices bis zu Action-Nodes. Jeder Node hat Input- und Output-Pins, über die der Flow verbunden wird. Nodes können Sub-Nodes (Requirements, SideEffects, Choices) enthalten.

---

### Participant

Eine `UMayDialogueParticipant`-Komponente, die an einen Actor gehängt wird. Sie identifiziert den Actor als Dialog-Teilnehmer über einen `ParticipantTag` und speichert Participant-Scope-Variablen. Ohne Participant-Komponente kann ein Actor keine Rolle im Dialog übernehmen.

---

### Participant-Scope

Variablen, die an einer Participant-Komponente hängen und über einzelne Dialoge hinaus überleben. Optional können Participant-Scope-Variablen in einem SaveGame gespeichert werden. Contrast: [Dialogue-Scope](#dialogue-scope).

---

### Player Choice

Ein Node-Typ, der dem Spieler eine oder mehrere Antwort-Optionen präsentiert. Der Dialog pausiert, bis der Spieler eine Choice auswählt. Jede Choice hat einen eigenen Output-Pin, der den Flow in die entsprechende Richtung weiterleitet.

---

### Preview Runner

`FMayDialoguePreviewRunner` — ein eingebettetes Playback-System im Asset-Editor. Spielt einen Dialog direkt im Editor ab, ohne PIE starten zu müssen. Ermöglicht schnelles Testen von Text, Struktur und Branch-Logik. Tags können simuliert werden, um Requirements zu testen.

Siehe: [Debug-Tipps → Preview-Runner](../troubleshooting/debugging-tips.md#preview-runner-iteration-ohne-pie).

---

### Requirement

Eine Bedingung, die erfüllt sein muss, damit ein Node oder eine Choice ausgeführt bzw. angezeigt wird. Requirements sind Sub-Nodes in Node-Bodies. Das Ergebnis einer Requirement-Prüfung ist `Passed`, `FailedButVisible` oder `FailedAndHidden`.

Eigene Requirement-Typen können als Blueprint-Subklasse von `UMayDialogueRequirement` erstellt werden.

---

### Sayline

Der häufigste Node-Typ. Eine SayLine zeigt einen Text eines Sprechers an, spielt optional Audio ab und wartet auf Advance (je nach Advance Mode). Die Title-Bar eines SayLine-Nodes hat die Farbe des zugewiesenen Sprechers.

---

### SideEffect

Ein Sub-Node, der eine Nebenwirkung ausführt, wenn ein Node betreten oder verlassen wird — z. B. eine Variable setzen, ein Event feuern, einen Tag hinzufügen. SideEffects haben kein Ergebnis, das den Flow beeinflusst.

---

### Speaker

`FMayDialogueSpeaker` — eine Asset-seitige Definition eines Sprechers. Enthält: Anzeigename, Portrait-Textur, Node-Farbe, Stimm-Asset-Mapping und Babel-Profil. Speaker werden im Speakers-Panel des Dialog-Assets definiert und per Speaker-Tag auf Nodes referenziert.

---

### SubGraph

Ein Teilgraph innerhalb desselben Dialog-Assets. SubGraphs ermöglichen es, wiederverwendbare Dialog-Abschnitte zu strukturieren, ohne ein eigenes Asset anlegen zu müssen. Betreten über einen SubGraph-Node, Rückkehr über dessen Exit.

---

### Subsystem

`UMayDialogueSubsystem` — die zentrale Welt-Instanz, die alle laufenden Dialoge verwaltet. Sie startet und stoppt Dialoge, löst Participants auf und stellt Delegates für Dialog-Events bereit. Zugriff über `GetWorld()->GetSubsystem<UMayDialogueSubsystem>()` oder den Blueprint-Funktionsknoten `GetMayDialogueSubsystem`.

---

### Typewriter

Ein optionaler Texteffekt, der SayLine-Text Zeichen für Zeichen aufdeckt. Konfigurierbar über `TypewriterCharsPerSecond` auf Speaker-Ebene. Babel-Stimmen sind mit dem Typewriter synchronisiert — Babel blippt bei jedem aufgedeckten Zeichen.

---

### Variables

Ein Satz deklarierter Schlüssel-Wert-Paare in einem Dialog-Asset. Variablen haben einen Typ (Bool, Int, Float, String, Tag), einen Scope (Dialogue oder Participant) und einen Defaultwert. Sie werden im Variables-Panel des Assets angelegt und über SetVariable-Nodes geschrieben sowie in Requirements ausgelesen.

---

> 📸 **Bild-Platzhalter:** `glossary-speakers-panel.png` — Speakers-Panel eines Dialog-Assets mit zwei definierten Sprechern.
> *Setup:* Dialog-Asset offen, Speakers-Tab aktiv. Tabelle zeigt zwei Einträge: Sprecher „Wächter" (Tag: `Dialogue.Speaker.Guard`, Farbe: dunkelrot, Portrait: Portrait-Textur zugewiesen) und Sprecher „Spieler" (Tag: `Dialogue.Speaker.Player`, Farbe: blau, Portrait: leer).
