# Quick Start – Dein erster Dialog in 5 Minuten

Diese Anleitung bringt dich vom leeren Projekt zu einem spielbaren NPC-Dialog im Level. Du brauchst nur das installierte Plugin (siehe [Installation](installation.md)).

## Was du erreichen wirst

Ein NPC steht im Level. Wenn der Spieler über eine Blueprint-Taste / Trigger den Dialog auslöst, spielt sich ein kurzer Text-Dialog mit zwei Antwort-Choices ab. Alles funktioniert **ohne eine einzige Zeile Blueprint-Logik für Text-Rendering, Audio oder Input**.

## Schritt 1 · Dialog-Asset anlegen

1. Content Browser → Rechtsklick in `Content/Dialogues/` (Ordner ggf. anlegen).
2. **Miscellaneous → MayDialogue Asset** wählen.
3. Asset benennen: `DA_Greeting_Simple`.
4. Doppelklick öffnet den Asset-Editor.

Du siehst den Graph mit einem **Entry-Node**. Der ist immer da und immer der Einstiegspunkt.

## Schritt 2 · Sprecher definieren

Im **Speakers-Panel** (rechts/Seitenleiste):

1. **Add Speaker** klicken.
2. Tag: `Dialogue.Speaker.Guard` – falls der Tag noch nicht existiert, wird er angelegt.
3. DisplayName: `Wächter`.
4. NodeColor: z.B. dunkelrot.

Ein zweiter Sprecher für den Spieler:

1. **Add Speaker** klicken.
2. Tag: `Dialogue.Speaker.Player`.
3. DisplayName: `Du`.
4. NodeColor: z.B. neutralgrau.

## Schritt 3 · Erste SayLine

1. Rechtsklick im Graph → **Say Line** wählen.
2. Node wird platziert, Title-Bar bekommt die Wächter-Farbe, sobald du…
3. …im Details-Panel den `SpeakerTag` auf `Dialogue.Speaker.Guard` setzt.
4. `DialogueText`: `"Halt! Wer bist du?"`.
5. Entry-Output-Pin mit dem Input-Pin der SayLine verbinden.

## Schritt 4 · Player-Choice

1. Rechtsklick im Graph → **Player Choice** wählen.
2. SayLine-Output mit PlayerChoice-Input verbinden.
3. `PromptText`: `"Du antwortest:"`.
4. Im Choices-Array **Add Element**, **zweimal**.
5. Choice 0: Text `"Ein Freund des Königs."`
6. Choice 1: Text `"Das geht dich nichts an."`

## Schritt 5 · Zwei Antworten für die Wahl

Für jede Choice brauchst du eine SayLine als Reaktion:

1. **Say Line 1** – SpeakerTag `Dialogue.Speaker.Guard`, Text: `"Dann passiere in Frieden."`
2. **Say Line 2** – SpeakerTag `Dialogue.Speaker.Guard`, Text: `"Dann verzieh dich!"`

Verbinde:

* Output-Pin 0 des PlayerChoice → SayLine 1.
* Output-Pin 1 des PlayerChoice → SayLine 2.

## Schritt 6 · Exit

1. Rechtsklick im Graph → **Exit** wählen. Einmal reicht.
2. Beide SayLines mit dem Exit verbinden.

Dein Graph sieht jetzt so aus:

```
[Entry] → [SayLine: "Halt!"] → [PlayerChoice]
                                 ├── Output 0 → [SayLine: "Passiere"] ─┐
                                 └── Output 1 → [SayLine: "Verzieh"] ──┤
                                                                     [Exit]
```

## Schritt 7 · Compile

**Toolbar → Compile**. Falls der Validator Fehler meldet, behebe sie laut Panel **Compiler Results**. Die häufigsten Fehler:

* Unverbundene Output-Pins → einen Output an Exit hängen.
* Sprecher-Tag leer → im Details-Panel setzen.
* Leere SayLines → Text eintragen.

## Schritt 8 · NPC im Level

1. Level öffnen.
2. Beliebigen Character / StaticMesh-Actor als **Wächter** platzieren.
3. Im Details-Panel des Actors: **Add Component → MayDialogue Participant**.
4. Komponente konfigurieren:
   * `ParticipantTag`: `Dialogue.Speaker.Guard`.
   * `DisplayName`: `Wächter`.
   * `DefaultDialogue`: `DA_Greeting_Simple` (das gerade erstellte Asset).

{% hint style="info" %}
Dein **Spieler-Pawn** braucht ebenfalls eine `MayDialogueParticipant`-Komponente mit `ParticipantTag = Dialogue.Speaker.Player`, damit SayLines für den Spieler korrekt adressiert werden können (auch wenn du im Quick Start keine spielerseitigen SayLines nutzt).
{% endhint %}

## Schritt 9 · Dialog starten

Im Blueprint-Graph deines Trigger-Actors (z.B. Box-Trigger oder Interact-Button):

**Variante A — direkt per Participant:**

1. Reference zum Wächter-Actor holen → **Get Component by Class: MayDialogueParticipant**.
2. Auf dem Participant: **Start Default Dialogue** aufrufen.
3. `Other`-Parameter: Referenz zur Spieler-Participant-Komponente (damit der Dialog einen Target kennt).

**Variante B — über die Library:**

```
MayDialogueLibrary.StartDialogue(
    WorldContext = Self,
    Asset        = DA_Greeting_Simple,
    Instigator   = PlayerPawn,
    Target       = GuardActor
)
```

## Schritt 10 · Testen

**PIE starten**. Geh zum Wächter, triggere den Dialog. Du solltest sehen:

* Ein Widget erscheint am Viewport (Slate-Debug-Widget, weil noch kein UMG zugewiesen).
* Der Wächter spricht: *"Halt! Wer bist du?"*.
* Zwei Choice-Buttons erscheinen.
* Klick auf eine Choice führt zur passenden Reaktion und beendet den Dialog.

{% hint style="success" %}
**Geschafft.** Du hast einen spielbaren Dialog ohne UMG, ohne Audio-Setup, ohne Input-Handling.
{% endhint %}

## Was als Nächstes?

* **Variablen & Bedingungen** kennenlernen → [Walkthrough](first-dialogue.md).
* **Das UMG durch eigene Widgets ersetzen** → [UI-Architektur](../ui/umg-architecture.md).
* **Audio hinzufügen** → [Audio-System](../audio/README.md).
* **Choices an GAS-Attribute knüpfen** → [GAS-Integration](../gas/README.md).

## Fehlerbehebung

<details>
<summary>Kein Widget erscheint beim Start</summary>

Prüfe in **Project Settings → Plugins → MayDialogue**, ob `bUseSlateDialogueWidget` aktiv ist und ob ein `DefaultDialogueWidgetClass` gesetzt ist. Standardmäßig erscheint das Slate-Debug-Widget auch ohne UMG.
</details>

<details>
<summary>Dialog startet nicht</summary>

Prüfe:

* Hat das Asset einen Entry-Node und **wurde es compiled**?
* Hat der Wächter-Actor eine `MayDialogueParticipant`-Komponente mit korrektem Tag?
* Hat der Spieler-Pawn ebenfalls einen Participant?
* Der Ausgabe-Log zeigt Log-Warnings, wenn etwas fehlt.
</details>

<details>
<summary>"Halt! Wer bist du?" wird gespielt, aber keine Antwort-Buttons</summary>

Das passiert, wenn der PlayerChoice-Node keine Outputs hat oder die Requirements aller Choices fehlschlagen (FailedAndHidden). Öffne das Asset, prüfe die Choices-Liste und verbinde ihre Output-Pins.
</details>
