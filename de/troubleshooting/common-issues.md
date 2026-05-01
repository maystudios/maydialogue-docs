---
description: Pro Symptom eine klare Ursache und eine konkrete Lösung.
---

# Häufige Probleme

Jedes Problem ist als Frage formuliert, damit du direkt das findest, was dich gerade blockiert.

---

## Dialog startet nicht

**Symptom:** Du rufst `StartDialogue` auf (z. B. im Interaktions-Trigger), aber nichts passiert. Kein Widget, kein Log-Eintrag, nichts.

**Häufige Ursachen und Lösungen:**

<details>
<summary>Asset nicht kompiliert</summary>

Das Asset enthält unsaved Compile-Errors. Öffne das Dialog-Asset, klicke **Compile** und behebe alle Validator-Fehler im Compiler-Results-Tab.

{% hint style="warning" %}
Ein Asset mit Compile-Error wird vom Subsystem abgelehnt, bevor eine Instance erzeugt wird.
{% endhint %}

</details>

<details>
<summary>Entry-Node fehlt</summary>

Jedes Dialog-Asset braucht genau einen **Entry**-Node. Fehlt er, gibt der Compiler `Asset has no entry point` aus. Füge den Entry-Node über die Palette hinzu und verbinde ihn mit dem ersten SayLine- oder Branch-Node.

</details>

<details>
<summary>Participant-Komponente fehlt oder Tag stimmt nicht</summary>

Jeder am Dialog beteiligte Actor braucht eine `UMayDialogueParticipant`-Komponente mit dem richtigen `ParticipantTag`.

- Spieler-Pawn: `ParticipantTag = Dialogue.Speaker.Player`
- NPC: `ParticipantTag` muss exakt dem SpeakerTag in den SayLines entsprechen.

Öffne den Actor im Details-Panel, suche die Komponente und prüfe den Tag-Wert.

</details>

<details>
<summary>Ein anderer Dialog läuft bereits</summary>

Das Subsystem erlaubt standardmäßig nur einen aktiven Dialog. Prüfe mit `MayDialogueSubsystem → IsAnyDialogueActive()`. Stoppe laufende Dialoge zuerst mit `StopAllDialogues()`.

</details>

> 📸 **Bild-Platzhalter:** `common-issues-start-participant.png` — Participant-Komponente im Details-Panel eines NPC-Actors.
> *Setup:* NPC-Blueprint im Editor öffnen. Im Components-Panel die `UMayDialogueParticipant`-Komponente auswählen. Im Details-Panel rechts ist sichtbar: `ParticipantTag`-Feld mit dem Wert `Dialogue.Speaker.Guard`. `DefaultDialogue`-Asset-Slot zeigt ein zugewiesenes Dialog-Asset.

---

## Widget erscheint nicht

**Symptom:** Im Output-Log siehst du, dass der Dialog gestartet hat (`StartDialogue: Instance created`), aber im Spiel oder PIE erscheint keine Dialogbox.

<details>
<summary>Kein Widget-Setup in den Project Settings</summary>

Gehe zu **Edit → Project Settings → MayDialogue → UI**. Prüfe:

- `DefaultDialogueWidgetClass` ist gefüllt (deine UMG-Klasse oder das mitgelieferte Standard-Widget).
- Alternativ: `bUseSlateDialogueWidget = true` für den eingebauten Slate-Fallback.

Ohne eine der beiden Optionen weiß das Plugin nicht, was es anzeigen soll.

</details>

<details>
<summary>UMG-Widget hat falsche BindWidget-Namen</summary>

Öffne dein UMG-Widget im Designer. Kritische Pflicht-Slots sind:

| Slot-Name | Typ |
| --- | --- |
| `DialogFrameWidget` | Border / Overlay |
| `TextWidget` | RichTextBlock |
| `ChoiceListWidget` | UMayDialogueWidget_ChoiceList |

Ein falscher oder fehlender Name erzeugt eine stille Warnung — kein Crash, aber kein sichtbares Widget.

</details>

<details>
<summary>Widget nach Level-Wechsel nicht neu gebunden (bekanntes Issue)</summary>

Das Static-Widget überlebt Level-Teardown nicht immer sauber. **Workaround:** Rufe `Subsystem → StopAllDialogues()` beim Level-Wechsel auf (z. B. im Level-Blueprint `BeginPlay` oder im `GameMode::HandleSeamlessTravel`), damit das Widget sich korrekt neu registriert.

</details>

> 📸 **Bild-Platzhalter:** `common-issues-widget-settings.png` — MayDialogue UI-Einstellungen in den Project Settings.
> *Setup:* Editor → Edit → Project Settings → MayDialogue-Kategorie → UI-Abschnitt. Sichtbar: `DefaultDialogueWidgetClass`-Dropdown mit ausgewählter Klasse `WBP_DialogueHUD`, `bUseSlateDialogueWidget`-Checkbox (deaktiviert).

---

## Choices fehlen

**Symptom:** Ein PlayerChoice-Node ist aktiv, aber im Spiel erscheinen keine Antwort-Buttons — oder nur ein Teil davon.

<details>
<summary>Alle Choices FailedAndHidden</summary>

Wenn jede Choice in einem PlayerChoice-Node eine Requirement hat, die fehlschlägt, **und** das Requirement als `FailedAndHidden` konfiguriert ist, verschwinden alle Buttons.

Teste im **Preview-Runner**: Setze Tags manuell, um Requirements zu simulieren. Wenn die Choices dort erscheinen, ist der GAS-Tag-State im PIE das Problem.

</details>

<details>
<summary>ChoiceButtonClass nicht gesetzt</summary>

Im `UMayDialogueWidget_ChoiceList`-Widget (Blueprint) muss die Property `ChoiceButtonClass` auf eine gültige Button-Klasse zeigen. Ist sie leer, werden keine Buttons erzeugt.

</details>

<details>
<summary>ChoiceListWidget-Slot fehlt im Top-Level-Widget</summary>

Das übergeordnete Dialogue-Widget muss einen `ChoiceListWidget`-BindSlot haben. Fehlt dieser, greift das Plugin auf einen Legacy-Pfad zurück, der keine Choices rendert.

</details>

> 📸 **Bild-Platzhalter:** `common-issues-choices-preview.png` — PlayerChoice-Node im Preview-Runner mit sichtbaren Choice-Buttons.
> *Setup:* Dialog-Asset mit PlayerChoice-Node öffnen, Preview-Runner starten. Sichtbar: das Preview-Panel zeigt eine SayLine des NPC und darunter zwei Choice-Buttons. Die erste Choice hat ein grünes Schloss-Icon (Requirement erfüllt), die zweite ein rotes Schloss-Icon (FailedButVisible).

---

## Audio fehlt

**Symptom:** Die SayLine erscheint als Text, aber es ist keine Stimme zu hören.

<details>
<summary>Voice-Asset für die aktuelle Culture fehlt</summary>

Prüfe im SayLine-Details-Panel den Slot `DialogueVoice`. Er ist nach Culture-Key aufgeteilt. Wenn deine aktive Culture kein zugewiesenes Audio-Asset hat, bleibt die SayLine stumm.

</details>

<details>
<summary>AudioComponent ist verdeckt oder Attenuation blockiert</summary>

Der Speaker-Actor spielt den Sound räumlich ab. Wenn der Spieler zu weit entfernt ist oder die Attenuation-Kurve zu aggressiv eingestellt ist, hört man nichts. Prüfe den `AttenuationOverride` an der Participant-Komponente oder setze `AudioModeOverride = Force2D` am Speaker für Nah-Dialog.

</details>

<details>
<summary>SoundClass ist gemutet</summary>

Prüfe unter **Edit → Project Settings → Audio** oder im Mixer, ob die `SoundClass` des Dialog-Assets (z. B. `SC_Voice`) auf voller Lautstärke ist.

</details>

> 📸 **Bild-Platzhalter:** `common-issues-audio-sayline.png` — SayLine-Node mit gefülltem Voice-Asset-Slot im Details-Panel.
> *Setup:* SayLine-Node im Dialog-Asset auswählen. Details-Panel zeigt: `SpeakerTag = Dialogue.Speaker.Guard`, `DialogueVoice`-Array mit einem Eintrag (Key: `en`, Value: zugewiesenes `SoundWave`-Asset). `AdvanceModeOverride = AfterVoice` ausgewählt.

---

## Babel stumm

**Symptom:** Die SayLine hat kein Voice-Asset, aber die prozedurale Babel-Stimme ist trotzdem nicht hörbar.

<details>
<summary>Babel nicht aktiviert</summary>

Gehe zu **Project Settings → MayDialogue → Audio**. `bEnableBabelVoice` muss `true` sein.

</details>

<details>
<summary>Kein BabelProfile zugewiesen</summary>

Am Speaker oder in den globalen Settings muss ein `DefaultBabelProfile` hinterlegt sein. Ohne Profil weiß Babel nicht, welchen Klang es erzeugen soll.

</details>

<details>
<summary>BlipSounds leer und ProceduralDefaults deaktiviert</summary>

Wenn `bUseProceduralDefaults = false` und kein `BlipSounds`-Array gefüllt ist, erzeugt Babel keinen Sound. Entweder Samples zuweisen oder `bUseProceduralDefaults = true` setzen.

</details>

<details>
<summary>OnCharacterRevealed nicht verbunden</summary>

Im Blueprint-Widget muss das Event `OnCharacterRevealed` des `TextWidget` mit `BabelSynth → OnCharacterRevealed` verbunden sein. Ohne diese Verbindung bekommt Babel kein Signal zum Abspielen.

</details>

---

## Variable nicht persistent

**Symptom:** Du setzt eine Variable mit einem SetVariable-Node, aber ein nachfolgender Requirement-Check oder ein zweiter Dialog liest noch den alten Wert.

<details>
<summary>Falscher Scope</summary>

Dialogue-Scope-Variablen existieren nur während des laufenden Dialogs. Wenn du eine Variable über mehrere Dialoge hinweg brauchst, verwende **Participant-Scope**.

| Scope | Lebensdauer |
| --- | --- |
| Dialogue-Scope | Nur während dieser Dialog-Instance läuft |
| Participant-Scope | Solange der Actor im Level existiert (optional SaveGame) |

</details>

<details>
<summary>SetVariable schreibt aktuell nur Dialogue-Scope (bekanntes Issue #13)</summary>

Der SetVariable-Node unterstützt im aktuellen Stand nur Dialogue-Scope. **Workaround:** Nutze einen Blueprint-SideEffect-Node, der direkt `Participant → SetPersistentVariable` aufruft.

</details>

<details>
<summary>Variable nicht im Variables-Panel deklariert</summary>

Öffne das Variables-Panel des Dialog-Assets. Die Variable muss dort mit korrektem Namen und Typ angelegt sein, bevor ein Node sie referenzieren kann.

</details>

<details>
<summary>Typ-Mismatch</summary>

Der Validator warnt bei inkonsistenten Typen (z. B. Bool-Variable mit String-Setter). Prüfe den Compiler-Results-Tab nach dem Compile.

</details>

> 📸 **Bild-Platzhalter:** `common-issues-variables-panel.png` — Variables-Panel eines Dialog-Assets mit mehreren deklarierten Variablen.
> *Setup:* Dialog-Asset öffnen, Variables-Tab aktivieren. Sichtbar: Tabelle mit Spalten Name, Typ, Scope, Defaultwert. Beispiel-Einträge: `bMetGuard` (Bool, Participant), `ChosenPassword` (String, Dialogue).

---

## Dialog bleibt hängen

**Symptom:** Der Dialog beginnt, zeigt eine SayLine, und dann passiert nichts mehr. Kein Advance, kein Ende.

<details>
<summary>Advance-Mode = AfterVoice, aber kein Voice-Asset</summary>

Der Dialog wartet auf den Voice-End-Callback, der nie kommt, weil kein Audio-Asset hinterlegt ist. Lösung: Voice-Asset zuweisen, oder `AdvanceModeOverride` auf `Manual` oder `Timer` ändern.

</details>

<details>
<summary>Wait-Node wartet auf Event, das nie gefeuert wird</summary>

Ein Wait-Node mit `WaitEventTag` blockiert so lange, bis das Tag-Event vom Subsystem empfangen wird. Prüfe, ob das Event irgendwo im Blueprint tatsächlich gefeuert wird (`Subsystem → FireDialogueEvent(Tag)`).

</details>

<details>
<summary>PlayAnimation mit bWaitForMontageEnd = true, Montage existiert nicht</summary>

Wenn das Montage-Asset leer oder ungültig ist, wird der End-Delegate nie gefeuert. Setze `bWaitForMontageEnd = false` oder weise ein gültiges Montage-Asset zu.

</details>

**Diagnose mit dem Debugger:** Setze einen Breakpoint auf den Node vor dem Problem, starte PIE, löse den Dialog aus. Wenn der Breakpoint trifft, klicke **Step Over** und beobachte, welcher Node nicht weiterspringt.

> 📸 **Bild-Platzhalter:** `common-issues-debugger-breakpoint.png` — Dialog-Debugger im Editor mit aktivem Breakpoint auf einem SayLine-Node.
> *Setup:* Dialog-Asset öffnen, Debugger-Tab aktivieren. Auf einem SayLine-Node einen roten Breakpoint-Punkt setzen (Rechtsklick → Toggle Breakpoint). PIE aktiv, Dialog ausgelöst. Sichtbar: Der Node leuchtet orange/gelb als aktuell aktiver Node, der Debugger-Tab zeigt die aktuellen Variablen-Werte.

---

## Compile-Fehler

**Symptom:** Der Compiler-Results-Tab zeigt einen Error, aber der Klick auf den Fehler springt auf einen falschen oder unklaren Node.

<details>
<summary>Andere Assets haben unsaved Changes mit Cross-References</summary>

Speichere alle offenen Dialog-Assets (`Ctrl+Shift+S` für alle) und kompiliere erneut.

</details>

<details>
<summary>Engine-Neustart nach Plugin-Änderungen</summary>

Nach größeren Änderungen am Plugin-Code (z. B. neue Node-Klasse) müssen alle Assets neu kompiliert werden. Starte den Editor neu und öffne das Asset erneut.

</details>

---

## Sub-Graph-Rückkehr fehlt

**Symptom:** Ein Sub-Graph wird betreten und erreicht seinen Exit-Node, aber statt zum rufenden Graph zurückzukehren, endet der gesamte Dialog.

<details>
<summary>bReturnAfterExit nicht gesetzt</summary>

Der SubGraph-Node im Eltern-Graph hat eine Property `bReturnAfterExit`. Sie muss `true` sein. Standardwert sollte `true` sein — prüfe, ob er versehentlich deaktiviert wurde.

</details>

<details>
<summary>SubGraph-Entry fehlt im Unter-Asset</summary>

Wenn das aufgerufene Asset keinen Entry-Node hat, gibt der Validator eine Warnung aus. Der Rücksprung kann nicht sauber aufgelöst werden. Entry-Node hinzufügen und verknüpfen.

</details>

---

## „Other MayDialogue Participant is null"

**Symptom:** `StartDefaultDialogue(Other)` schlägt fehl mit einer Log-Warnung über einen leeren `Other`-Parameter.

Der `Other`-Actor hat keine `UMayDialogueParticipant`-Komponente, oder sie wurde noch nicht angelegt (z. B. bei dynamisch gespawnten Actors).

**Lösung:** Füge die Komponente als Default-Component im Blueprint-Defaults-Panel des Actors hinzu. Bei dynamisch gespawnten Actors: `AddComponentByClass<UMayDialogueParticipant>` in `BeginPlay`.

> 📸 **Bild-Platzhalter:** `common-issues-participant-beginplay.png` — Blueprint-Graph mit AddComponent-Aufruf in BeginPlay.
> *Setup:* NPC-Blueprint öffnen, Event-Graph → BeginPlay. Sichtbar: `Event BeginPlay` → `Add Component by Class` (Component Class: `UMayDialogueParticipant`) → `Set Participant Tag` (Tag: `Dialogue.Speaker.Guard`). Pins vollständig verbunden.
