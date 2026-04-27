---
description: Sprecher anlegen, Farben wählen, Portraits setzen und Audio-Overrides konfigurieren.
---

# Speakers-Panel

Das Speakers-Panel ist der zentrale Ort, an dem du alle Gesprächspartner eines Dialog-Assets definierst. Jeder Sprecher erhält einen Tag, einen Anzeigenamen, eine Farbe und optional ein Portrait.

> 📸 **Bild-Platzhalter:** `speakers-panel-overview.png` — Speakers-Panel mit drei eingetragenen Sprechern.
> *Setup:* Speakers-Tab aktiv. Drei Zeilen sichtbar: „Guard" (dunkelrot, Portrait-Slot gefüllt), „Player" (grau, kein Portrait), „Narrator" (blaugrau). Alle Felder ausgefüllt – Tag, DisplayName, Portrait-Thumbnail, Farb-Chip.

## Sprecher hinzufügen

1. Speakers-Tab öffnen.
2. **Add Speaker** klicken.
3. **Tag** eingeben – Auto-Complete aus dem zentralen GameplayTags-Register.
4. **DisplayName** eintippen (der Name, der später im UI unter dem Portrait erscheint).
5. **Portrait** per Dropdown aus dem Asset-Browser wählen (optional).
6. **Farbe** per Color-Picker setzen.
7. Optional: **Audio-Overrides** aufklappen und anpassen.

> 📸 **Bild-Platzhalter:** `speakers-panel-add-speaker.png` — „Add Speaker"-Button angeklickt, neue leere Zeile erscheint, Cursor im Tag-Feld.
> *Setup:* Speakers-Panel mit einer bestehenden Guard-Zeile. Darunter eine neue, leere Zeile mit aktivem Tag-Eingabefeld und Auto-Complete-Dropdown offen. Roter Pfeil auf „Add Speaker"-Button.

## Felder im Detail

| Feld | Bedeutung |
| --- | --- |
| **Tag** | GameplayTag zur Verknüpfung mit dem Participant-Actor (z.B. `Dialogue.Speaker.Guard`) |
| **DisplayName** | Angezeigter Name im Dialog-UI |
| **Portrait** | Soft-Referenz auf eine Textur (wird nur bei Bedarf geladen) |
| **Farbe** | Titelleisten-Farbe aller SayLine-Nodes dieses Sprechers im Graph |
| **Audio Mode Override** | Wie Voice-Assets abgespielt werden (z.B. 2D/3D) |
| **Sound Class Override** | Abweichendes Sound-Class-Asset für diesen Sprecher |
| **Attenuation Override** | Eigene Attenuation-Settings (z.B. für einen Geist: leisere Spatialisierung) |
| **Volume Multiplier** | Relative Lautstärke (1.0 = Projekt-Default) |
| **Pitch Multiplier** | Relative Tonhöhe (1.0 = unverändert) |
| **Babel Profile** | Optionales Profil für synthetisierte Sprache |

## Farben wählen – praktische Hinweise

Die Farbe eines Sprechers ist die **Titelleisten-Farbe** seiner SayLine-Nodes im Graph. Gut gewählte Farben machen einen 40-Node-Dialog sofort lesbar – du erkennst auf einen Blick, wer gerade spricht.

{% hint style="success" %}
**Tipp:** Verwende dieselben Farben konsistent für denselben Charakter über alle Assets hinweg. Lege die Farb-Codes einmalig in einem zentralen Dokument oder Design-System fest.
{% endhint %}

Bewährte Farbwahl:
- Spieler-Charakter: Neutrales Grau (`#808080`)
- Antagonist: Dunkelrot (`#8B0000`) oder Dunkelviolett
- Narrator: Dunkelblau oder Schwarz
- NPC-Freund: Gedämpftes Grün oder Blaugrau

> 📸 **Bild-Platzhalter:** `speakers-panel-color-picker.png` — Color-Picker offen, Guard-Sprecher in dunkelrot eingestellt.
> *Setup:* Farb-Chip des Guard-Eintrags angeklickt. UE-Color-Picker offen, Farbe `#8B0000` eingestellt. Im Hintergrund sieht man im Graph, wie ein SayLine-Node sofort die neue Farbe in der Titelleiste zeigt.

## Sprecher ändern

Alle Felder sind live editierbar. Ändere die **Farbe** eines Sprechers – alle SayLine-Nodes dieses Sprechers im Graph aktualisieren ihre Titelleiste sofort.

## Sprecher löschen

**Remove**-Button in der Sprecher-Zeile. Wenn noch SayLine-Nodes den gelöschten Tag referenzieren, meldet der Validator beim nächsten Compile einen **Missing Speaker**-Error.

## Speaker-Tag-Konventionen

Empfohlenes Format: `Dialogue.Speaker.<Name>` oder `Dialogue.Speaker.<Rolle>`

Beispiele:
- `Dialogue.Speaker.Player`
- `Dialogue.Speaker.Guard`
- `Dialogue.Speaker.Narrator`
- `Dialogue.Speaker.InnerVoice`

{% hint style="info" %}
Der Tag muss mit dem `ParticipantTag` des zugehörigen NPC-Actors übereinstimmen. Liegt keine Übereinstimmung vor, kann der Dialog-Actor den Sprecher zur Laufzeit nicht auflösen.
{% endhint %}

## Audio-Overrides: Wann nutzen?

Lass die Audio-Override-Felder **leer**, wenn der Sprecher denselben Standard-Klang hat wie der Rest des Projekts. Setze sie nur, wenn ein Sprecher sich klanglich unterscheiden soll:

- Geist: Attenuation und Volume auf halbe Stärke
- Radiogeräusch: Pitch Multiplier leicht erhöht, dediziertes Sound-Class
- Narrator (off-screen): 2D Audio Mode Override

> 📸 **Bild-Platzhalter:** `speakers-panel-audio-overrides.png` — Audio-Overrides ausgeklappt für einen Geist-Sprecher.
> *Setup:* Zeile „Ghost" mit aufgeklapptem Audio-Overrides-Abschnitt. Volume Multiplier auf 0.4, Attenuation-Override gefüllt, Sound-Class-Override gefüllt. Alle anderen Felder leer. Vergleich: Guard-Zeile darunter mit komplett leeren Audio-Feldern.

## Speaker-Dropdown im Details-Panel

Sobald du Sprecher im Panel angelegt hast, erscheint beim `SpeakerTag`-Feld in den Node-Details ein **Dropdown** statt eines generischen Tag-Pickers. Das Dropdown zeigt:

- Farb-Chip des Sprechers
- DisplayName
- Tag (als Tooltip)

Das spart Zeit – du tippst keine Tag-Pfade, du wählst aus einer kurzen Liste.

> 📸 **Bild-Platzhalter:** `speakers-dropdown-in-details.png` — Details-Panel einer SayLine mit geöffnetem Speaker-Dropdown.
> *Setup:* SayLine-Node selektiert. Details-Panel zeigt SpeakerTag-Feld als Dropdown, aufgeklappt mit zwei Einträgen: Guard (dunkelroter Chip, „Wächter") und Player (grauer Chip, „Du"). Roter Pfeil auf das Dropdown.

Weiter: [Variables-Panel →](variables-panel.md)
