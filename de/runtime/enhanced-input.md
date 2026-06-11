---
description: Dialog-Eingaben über Enhanced Input steuern — Tastatur, Gamepad und Touch, vollständig remappbar.
---

# Enhanced Input Integration

MayDialogue spricht **Enhanced Input** — UE5s modernes, datengetriebenes Eingabesystem. Damit definierst du deine Dialog-Steuerung (Weiter, Skip, Abbrechen, Auswahl wählen) als `UInputAction`-Assets, machst sie projektweit remappbar und lässt das Plugin automatisch erkennen, ob der Spieler gerade Tastatur oder Gamepad nutzt — der Skip-Button-Glyph und der Slate-„Weiter"-Hinweis passen sich von selbst an.

Du musst dafür nichts zwingend einrichten: Lässt du Enhanced Input aus oder belegst keine Actions, greift MayDialogue auf fest verdrahtete Fallback-Tasten zurück und funktioniert sofort.

{% hint style="info" %}
Diese Seite löst den manuellen `SetInputDevice`-Pfad aus dem [Skip-Button-Kapitel](../ui/skip-button.md) als empfohlenen Weg ab. `SetInputDevice` bleibt als **manueller Override** erhalten — du kannst die Geräteerkennung jederzeit selbst übersteuern.
{% endhint %}

## Sofort lauffähig: die Fallback-Tasten

Ohne jede Einrichtung reagiert der Dialog bereits auf sinnvolle Standardtasten. Diese Belegung greift immer, wenn Enhanced Input deaktiviert ist **oder** eine Action-Slot leer bleibt:

| Aktion | Tastatur | Gamepad |
| --- | --- | --- |
| Weiter / Skip Typewriter | `Leertaste`, `Enter` | `(A)` |
| Auswahl nach oben | `Pfeil ↑`, `W` | `D-Pad ↑` |
| Auswahl nach unten | `Pfeil ↓`, `S` | `D-Pad ↓` |
| Auswahl bestätigen | `Enter`, `Leertaste` | `(A)` |
| Auswahl direkt wählen | `1` – `9` | — |
| Dialog abbrechen | `Esc` | `(B)` |

Für einen Game-Jam oder Prototyp reicht das vollständig aus. Wer Eingaben remappbar machen oder eigene Tasten vergeben will, richtet Enhanced Input ein — der nächste Abschnitt zeigt das Schritt für Schritt.

## Schritt für Schritt: IA- und IMC-Assets anlegen

### Schritt 1 — Input Action Assets erstellen

Lege für jede Dialog-Aktion ein `Input Action`-Asset an (**Rechtsklick im Content Browser → Input → Input Action**). Für ein vollständiges Setup brauchst du:

```text
IA_Dialogue_Advance        (Digital / bool)
IA_Dialogue_Skip           (Digital / bool)
IA_Dialogue_Abort          (Digital / bool)
IA_Dialogue_ChoiceUp       (Digital / bool)
IA_Dialogue_ChoiceDown     (Digital / bool)
IA_Dialogue_ChoiceConfirm  (Digital / bool)
IA_Dialogue_Choice1 … 9    (Digital / bool — optional, für Direktwahl)
```

Alle sind vom Value-Type **Digital (bool)** — es sind einfache Tastendrücke.

### Schritt 2 — Mapping Context anlegen

Erstelle einen `Input Mapping Context` (**Rechtsklick → Input → Input Mapping Context**), z.B. `IMC_Dialogue`. Trage darin pro Action die gewünschten Tasten ein:

```text
IMC_Dialogue
 ├─ IA_Dialogue_Advance      → Space, Gamepad Face Button Bottom
 ├─ IA_Dialogue_Abort        → Escape, Gamepad Face Button Right
 ├─ IA_Dialogue_ChoiceUp     → Up, W, Gamepad D-Pad Up
 ├─ IA_Dialogue_ChoiceDown   → Down, S, Gamepad D-Pad Down
 ├─ IA_Dialogue_ChoiceConfirm→ Enter, Gamepad Face Button Bottom
 └─ IA_Dialogue_Choice1…9    → 1 … 9
```

> 📸 **Bild-Platzhalter:** `enhanced-input-imc-editor.png` — Input-Mapping-Context-Editor mit den Dialog-Actions und ihren Key-Bindings.
> *Setup:* `IMC_Dialogue` im Asset-Editor geöffnet. Mappings-Liste sichtbar: `IA_Dialogue_Advance` mit Keys „Space" und „Gamepad Face Button Bottom", darunter `IA_Dialogue_Abort` mit „Escape" und „Gamepad Face Button Right". Mindestens drei Actions ausgeklappt, damit die Key-Zuordnung klar erkennbar ist.

### Schritt 3 — In den Projekteinstellungen zuweisen

Öffne **Edit → Project Settings → Plugins → MayDialogue**, Kategorie **Dialogue | Enhanced Input**, und trage deine Assets ein:

```text
Use Enhanced Input          = true        (Standard)
Dialogue Mapping Context     = IMC_Dialogue
Mapping Context Priority     = 100         (Standard)
Auto Detect Input Device     = true        (Standard)

Advance Action               = IA_Dialogue_Advance
Skip Action                  = IA_Dialogue_Skip
Abort Action                 = IA_Dialogue_Abort
Choice Up Action             = IA_Dialogue_ChoiceUp
Choice Down Action           = IA_Dialogue_ChoiceDown
Choice Confirm Action        = IA_Dialogue_ChoiceConfirm
Choice By Index Actions       = [ IA_Dialogue_Choice1 … IA_Dialogue_Choice9 ]
```

Das war's. Sobald ein Dialog startet, fügt MayDialogue den `DialogueMappingContext` mit der angegebenen Priorität hinzu und entfernt ihn beim Dialog-Ende wieder.

> 📸 **Bild-Platzhalter:** `enhanced-input-project-settings.png` — Project Settings → Plugins → MayDialogue, Kategorie „Dialogue | Enhanced Input" mit gesetzten Enhanced-Input-Feldern.
> *Setup:* Project-Settings-Fenster, Sektion MayDialogue, zur Input-Kategorie gescrollt. `Use Enhanced Input`-Checkbox aktiv, `Dialogue Mapping Context` zeigt `IMC_Dialogue`, die Action-Slots (`Advance Action`, `Abort Action`, …) sind mit den IA-Assets befüllt. Roter Rahmen um die Enhanced-Input-Felder.

## Automatische Geräteerkennung

Ist `bAutoDetectInputDevice = true` (Standard), beobachtet das Plugin, welches Eingabegerät zuletzt benutzt wurde, und aktualisiert die UI entsprechend:

* Der **Skip-Button** zeigt den passenden Glyph (Leertaste-Symbol bei Tastatur, `(A)` bei Gamepad).
* Der **Slate-„Weiter"-Hinweis** im Debug-Widget zeigt denselben Geräte-Hinweis.

Du musst dafür nichts binden — die Erkennung speist `SkipButton` und Slate-Hint automatisch. Wechselt der Spieler mitten im Gespräch von Tastatur auf Controller, springt der Glyph live um.

{% hint style="info" %}
Brauchst du volle Kontrolle (z.B. weil dein Projekt eine eigene Geräteerkennung hat), setze `bAutoDetectInputDevice = false` und ruf wie bisher `SetInputDevice(bool bIsGamepad)` am Skip-Button auf. Der manuelle Override hat dann Vorrang.
{% endhint %}

## Remapping-Rezept: Spieler lässt Tasten umbelegen

Weil die Dialog-Steuerung über Enhanced Input läuft, bekommst du Remapping fast geschenkt. Nutze UEs `Enhanced Input User Settings` (Player Mappable Keys):

1. Markiere in deinem `IMC_Dialogue` die Mappings, die der Spieler ändern darf, als **Player Mappable** (gib jedem einen `Mapping Name`, z.B. `Dialogue_Advance`).
2. Baue ein Settings-Widget, das `Map Player Key In Slot` auf den `Enhanced Input User Settings` aufruft, wenn der Spieler eine neue Taste drückt.
3. Fertig — die neue Belegung gilt sofort auch im Dialog, da MayDialogue dieselben Actions liest.

```text
[Spieler drückt neue Taste im Options-Menü]
    │
    ▼
[Map Player Key In Slot]   (Enhanced Input User Settings)
    ├─ Mapping Name: "Dialogue_Advance"
    └─ New Key:      (erfasste Taste)
         → ab jetzt advanced der Dialog mit der neuen Taste
```

> 📸 **Bild-Platzhalter:** `enhanced-input-remap-menu.png` — In-Game-Optionsmenü mit einer remappbaren „Dialog weiter"-Zeile.
> *Setup:* Einfaches Settings-Widget im PIE. Zeile „Dialog: Weiter" mit aktueller Taste „Space" und einem „Neu belegen"-Button. Daneben (oder als zweites Bild) der Moment nach dem Umbelegen, Taste zeigt jetzt „E".

## API-Referenz: Einstellungsfelder

Alle Felder liegen unter `UMayDialogueSettings`, Kategorie **Dialogue | Enhanced Input** (auch `bAutoDetectInputDevice` liegt in derselben Kategorie). Jedes Feld außer `bUseEnhancedInput` hat eine `EditCondition` auf `bUseEnhancedInput`, ist also ausgegraut, solange der Master-Schalter aus ist.

| Feld | Typ | Standard | Zweck |
| --- | --- | --- | --- |
| `bUseEnhancedInput` | `bool` | `true` | Master-Schalter. Aus = nur die Fallback-Tasten greifen. |
| `DialogueMappingContext` | `TSoftObjectPtr<UInputMappingContext>` | *(leer)* | IMC, das beim Dialog-Start hinzugefügt und beim Ende entfernt wird. |
| `MappingContextPriority` | `int32` | `100` | Priorität, mit der der IMC gepusht wird (höher = überlagert niedrigere Kontexte). |
| `bAutoDetectInputDevice` | `bool` | `true` | Erkennt Tastatur/Gamepad automatisch und speist Skip-Glyph + Slate-Hint. |
| `AdvanceAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | „Weiter" / Typewriter-Skip. |
| `SkipAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Voice-/Zeilen-Skip (falls separat vom Advance gewünscht). |
| `AbortAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Dialog abbrechen. |
| `ChoiceUpAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Auswahl-Cursor nach oben. |
| `ChoiceDownAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Auswahl-Cursor nach unten. |
| `ChoiceConfirmAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Markierte Auswahl bestätigen. |
| `ChoiceByIndexActions` | `TArray<TSoftObjectPtr<UInputAction>>` | *(leer)* | Direktwahl: Index 0 = Auswahl 1, Index 1 = Auswahl 2, … Bis zu neun Einträge möglich (spiegelt die fest verdrahteten 1–9-Zifferntasten). |

{% hint style="info" %}
Alle Action-Slots sind `TSoftObjectPtr` — leere Slots **fallen einzeln** auf ihre Fallback-Taste zurück. Du kannst also z.B. nur `AdvanceAction` und `AbortAction` belegen und die Choice-Navigation den Standardtasten überlassen.
{% endhint %}

## Stolperfallen

| Fall | Was passiert / zu beachten |
| --- | --- |
| **Enhanced-Input-Plugin nicht aktiv** | Ist das engine-seitige Enhanced-Input-Plugin im Projekt deaktiviert, greifen nur die Fallback-Tasten — unabhängig von `bUseEnhancedInput`. |
| **IMC gesetzt, aber Actions leer** | Der Kontext wird zwar gepusht, aber ohne belegte Actions tut sich nichts Dialog-spezifisches → die leeren Slots fallen auf die Fallback-Tasten zurück. |
| **`bSwitchToUIInputDuringDialogue` interagiert** | Während des Dialogs schaltet das Plugin standardmäßig in den Game+UI-Input-Modus (Maus sichtbar, siehe [Projekteinstellungen](../getting-started/project-settings.md)). Stelle sicher, dass dein IMC in diesem Modus weiterhin Eingaben erhält. |
| **Geräteerkennung springt zu oft** | Manche Setups feuern Phantom-Eingaben (z.B. ein angeschlossener, aber ungenutzter Controller). Setze `bAutoDetectInputDevice = false` und treibe den Glyph manuell über `SetInputDevice`. |
| **Choice-Direktwahl über 9 hinaus** | `ChoiceByIndexActions` deckt 1–9 ab. Auswahllisten mit mehr Einträgen navigierst du über Up/Down/Confirm. |
