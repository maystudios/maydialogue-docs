# Speakers-Panel

Das Speakers-Panel ist der zentrale Ort für **alle Sprecher eines Dialog-Assets**. Hier definierst du, wie jeder Sprecher im UI und in der Audio-Pipeline präsentiert wird.

## Anatomie

Pro Sprecher eine Zeile mit:

| Spalte | Bedeutung |
| --- | --- |
| **Tag** | `FGameplayTag`, zum Matching mit `UMayDialogueParticipant::ParticipantTag`. |
| **DisplayName** | `FText`, wird im UI unter dem Portrait gezeigt. |
| **Portrait** | `TSoftObjectPtr<UTexture2D>`, wird asynchron geladen. |
| **Farbe** | `FLinearColor`, Title-Bar-Farbe der SayLine-Nodes dieses Sprechers. |
| **Audio-Overrides** | `AudioModeOverride`, `SoundClassOverride`, `AttenuationOverride`, `VolumeMultiplier`, `PitchMultiplier`. |
| **BabelProfile** | `TSoftObjectPtr<UMayDialogueBabelProfile>`. |

## Workflow

### Sprecher hinzufügen

1. Panel öffnen.
2. **Add Speaker** klicken.
3. Tag setzen (Auto-Complete aus GameplayTags-Hierarchie).
4. DisplayName eintippen.
5. Portrait per Drop-Down aus dem Asset-Browser.
6. Farbe per Color-Picker.
7. Optional: Audio-Overrides ausklappen und setzen.

### Sprecher ändern

Alle Felder sind Live-Edit. Änderungen am `NodeColor` werden sofort im Graph sichtbar (alle SayLines dieses Sprechers ändern ihre Title-Bar-Farbe).

### Sprecher löschen

**Remove**-Button in der Zeile. Wenn noch Nodes auf den Tag verweisen, warnt der Validator beim nächsten Compile (*„Missing Speaker"*).

## Best Practices

### Tag-Konvention

Empfehlung: `Dialogue.Speaker.<Name>` oder `Dialogue.Speaker.<Role>`:

* `Dialogue.Speaker.Player`
* `Dialogue.Speaker.Guard`
* `Dialogue.Speaker.Narrator`
* `Dialogue.Speaker.InnerVoice`

### Konsistente Farben

Im selben Spiel sollten dieselben Charaktere überall dieselben Farben haben. Trage die Farbe einmal in `DefaultGame.ini` als Konvention ein, oder leite sie aus einem zentralen Charakter-DataTable ab.

### Portraits als Soft-References

`TSoftObjectPtr` bedeutet: Das Portrait wird erst geladen, wenn der Sprecher auftritt. Das hält die Asset-Größe klein und startet Level schneller. Nicht nötig, Portraits als Hard-Reference anzulegen.

### Audio-Overrides nur für Ausnahmen

Wenn ein Sprecher denselben Sound-Klang hat wie der Projekt-Default, **lass die Override-Felder leer**. Nur wenn der Geist leiser klingen soll als der Detektiv, setzt du die Attenuation am Speaker (oder pro Node).

## Speaker-Customization im Details-Panel

Das Editor-Modul registriert eine **Details-Panel-Customization** für alle `FGameplayTag`-Felder, die Sprecher referenzieren. Das heißt:

* In den Properties einer SayLine erscheint beim `SpeakerTag` **kein** generischer Tag-Picker.
* Stattdessen ein Dropdown mit **allen im aktuellen Asset definierten Sprechern**, inklusive Farb-Chip und DisplayName.

Das macht den Workflow deutlich schneller – du tippst nicht Tag-Pfade, du wählst Sprecher aus einer kurzen Liste.

## Warnungen im Validator

Der Validator prüft:

* **Missing Speaker**: Ein Node referenziert einen Tag, der **nicht** im Speakers-Panel steht.
* **Unused Speaker** (Warning): Ein Speaker ist definiert, wird aber von keinem Node genutzt.

## Beispiel

Ein minimales Speakers-Panel für den Quick-Start-Dialog:

| Tag | DisplayName | Portrait | Farbe |
| --- | --- | --- | --- |
| `Dialogue.Speaker.Guard` | Wächter | P_Guard | `#8B0000` |
| `Dialogue.Speaker.Player` | Du | *(keins)* | `#808080` |

Weiter: [Variables-Panel →](variables-panel.md).
