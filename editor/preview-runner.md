# Preview Runner – ohne PIE testen

Der **Preview-Runner** ist die wahrscheinlich beliebteste Editor-Funktion. Er spielt einen Dialog **direkt im Asset-Editor** – ohne PIE, ohne Level, ohne Spieler-Spawn. Text, Audio, Babel, Choices, Variable-Simulation – alles im Editor-Fenster.

## Wozu?

Dialog-Autoring ist iterativ. Eine Zeile ändern, sofort hören, wie sie klingt. Eine Choice-Reihenfolge umstellen, sofort testen, wie sich die Entscheidung liest. Wenn zwischen jedem Test eine PIE-Ladung steht, stirbt die Kreativität.

Der Preview-Runner schneidet die Iterationszeit auf **unter eine Sekunde**.

## Panel-Aufbau

Der **Preview-Tab** ist in drei Bereiche geteilt:

1. **Dialog-Display** (oben) – zeigt den aktuellen Sprecher, seinen Text, seine Choices.
2. **Steuerung** (mittig) – Play / Stop / Advance / Choice-Buttons.
3. **Preview-State** (unten) – simulierte Tags / Attribute / Abilities, Variable-Liste, Event-Log.

## Starten

1. Toolbar-Button **Play Dialog** (alternativ: Icon im Preview-Panel).
2. Der Runner erzeugt eine temporäre `UMayDialogueInstance`-ähnliche Struktur in Preview-Memory (kein echter Subsystem-Call).
3. Die Ausführung startet am Entry-Node.

## Ablauf

Der Runner respektiert alle Advance-Modi und Node-Typen:

* **SayLine** wird mit Typewriter-Effekt angezeigt.
* **Voice-Assets** werden 2D abgespielt (mit Culture-Auswahl, siehe unten).
* **Babel** wird bei SayLines ohne Voice synthetisiert.
* **PlayerChoice** zeigt anklickbare Buttons.
* **Branch**, **RandomLine**, **Wait**, **Link**, **SubGraph** funktionieren wie in Produktion.

Action-Nodes (CameraFocus, CameraShake, PlayAnimation, ApplyEffect) werden im Preview **geloggt, aber nicht echt ausgeführt** – es gibt ja keine Welt und keinen Actor, an dem sie wirken könnten.

## Simulierter GAS-State

Für jedes Dialog-Asset scannt der Runner die verwendeten Tags, Attribute, Abilities und listet sie im Preview-State-Bereich auf. Du kannst sie **manuell umschalten**:

* **Tags** – Checkbox-Liste. Aktivierte Tags simulieren das Vorhandensein am Spieler-ASC.
* **Attributes** – Number-Fields. Ändere den simulierten Wert.
* **Abilities** – Checkbox-Liste. Simuliert ob der Spieler die Ability hat.

Sobald du einen Wert änderst, werden Requirement-Pills auf den Choice-Sub-Nodes live neu evaluiert und ihre Farbe/Sichtbarkeit angepasst.

{% hint style="info" %}
**Aktuelle Einschränkung**: Die Live-Requirement-Evaluation für Choice/Branch-Pills ist noch als TODO markiert (`SMayDialogueGraphNode_Task.cpp`). Die Pill-Farben zeigen derzeit den statischen `bHideOnFail`-Wert. Die Runtime-Logik im Preview funktioniert trotzdem korrekt – nur die visuellen Pill-Farben sind nicht live.
{% endhint %}

## Variable-Watch

Der Preview-State-Bereich listet alle Dialog-Scope-Variablen in Echtzeit. Du kannst ihre Werte live ändern, um Branches durchzutesten.

## Culture-Switch

Das Dropdown **Culture** oben im Preview-Panel wechselt zur Laufzeit die aktive Culture:

1. SayLine-Voice-Assets werden aus `VoicePerCulture[NewCulture]` neu aufgelöst.
2. FText-Werte aktualisieren sich automatisch (UE's Lokalisierungssystem).
3. Babel-Synthese wird mit dem neuen Text angestoßen.

Perfekt für Lokalisierungs-Tests.

## Steuerung

| Button | Wirkung |
| --- | --- |
| **Play** | Preview starten. |
| **Stop** | Preview beenden. Audio stoppt, State wird resetted. |
| **Advance** | Im WaitingForAdvance-Zustand: nächsten Node ansteuern. |
| **Skip Typewriter** | Text sofort auf Ende. |
| **Choice-Buttons** | In WaitingForChoice-Zustand: Auswahl triggern. |

## Event-Log

Das untere Panel zeigt eine zeitgeordnete Liste:

* Node-Transitions (*„Node A → Node B"*).
* Variable-Changes.
* FireEvent-Auslösungen.
* Fehler (Requirement-Failures, Invalid-Links).

## Wann du NICHT den Preview-Runner nutzen solltest

* Wenn dein Dialog echte Welt-Interaktion braucht (NPCs reagieren auf Physics, AI-Behavior-Zustände).
* Wenn Kamera-Schwenks, Animationen oder echte GameplayEffects ein kritischer Teil des Gefühls sind.
* Wenn du Multiplayer-Pfade testen willst.

In diesen Fällen: **PIE + Debugger**.

Weiter: [Debugger & Breakpoints →](debugger.md).
