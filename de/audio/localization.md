---
description: Für jede Sprache ein eigenes Voice-Asset – wie die VoicePerCulture-Map funktioniert und wie das System zur Laufzeit auflöst.
---

# Lokalisierung (VoicePerCulture)

## Wann brauche ich das?

Wenn dein Spiel in mehreren Sprachen erscheint und du pro Sprache eigene Voice-Aufnahmen hast:

- Englische Stimme für `en`, deutsche für `de`, japanische für `ja`
- Zur Laufzeit spielt automatisch die passende Aufnahme – ohne Blueprint-Logik

MayDialogue kombiniert das mit UEs Standard-Text-Lokalisierung: der **Text** einer SayLine wird über das normale UE-Lokalisierungs-Dashboard übersetzt, die **Stimme** über die `DialogueVoice`-Map auf der SayLine.

## Die VoicePerCulture-Map

Auf jeder SayLine gibt es eine Map:

```text
DialogueVoice:
  "en" → VO_EN_Hello
  "de" → VO_DE_Hallo
  "ja" → VO_JA_Konnichiwa
```

Key: BCP-47-Culture-Code. Value: das `USoundBase`-Asset für diese Kultur.

> 📸 **Bild-Platzhalter:** `localization-voiceperculture-map.png` — SayLine-Details-Panel mit ausgefüllter DialogueVoice-Map.
> *Setup:* SayLine-Node im Dialog-Editor auswählen. Details-Panel rechts: Bereich "Audio" aufgeklappt, `DialogueVoice`-Map sichtbar mit drei Einträgen: Key `en` → Asset `VO_EN_Hello`, Key `de` → Asset `VO_DE_Hallo`, Key `ja` → Asset `VO_JA_Konnichiwa`. Roter Pfeil auf die Map-Einträge.

## Wie du Einträge setzt

1. SayLine-Node im Graph auswählen
2. Details-Panel → Bereich "Audio" → `DialogueVoice`
3. `+`-Button → Key eingeben (z.B. `"de"`) → Asset-Slot mit Voice-Asset füllen
4. Pro Sprache wiederholen

> 📸 **Bild-Platzhalter:** `localization-add-entry.png` — DialogueVoice-Map mit dem "+" Button und einem halb ausgefüllten neuen Eintrag.
> *Setup:* Gleiche Ansicht wie oben, aber Fokus auf den `+`-Button und einen neu angelegten, noch leeren Key-Value-Eintrag. Key-Feld ist aktiv (Cursor sichtbar), Value-Slot noch leer. Zeigt den Arbeitsablauf des Hinzufügens.

## Auflösungs-Logik zur Laufzeit

MayDialogue fragt UEs Locale-API nach der aktuellen Culture und löst so auf:

1. Suche `DialogueVoice[aktuelle Culture]` — z.B. `"de-AT"`
2. Nicht gefunden? → Region abschneiden, suche `"de"`
3. Nicht gefunden? → Suche Default-Key (typisch `"en"` oder leerer String)
4. Immer noch nicht gefunden? → Kein Voice-Asset → Babel oder Stille

```text
Gesuchte Culture: "de-AT"
  → Lookup "de-AT" → nicht gefunden
  → Lookup "de"    → gefunden: VO_DE_Hallo ✓
```

## Gängige Culture-Codes

| Code | Sprache |
|---|---|
| `en` | Englisch |
| `de` | Deutsch |
| `fr` | Französisch |
| `ja` | Japanisch |
| `zh-Hans` | Chinesisch (Vereinfacht) |
| `es-419` | Spanisch (Lateinamerika) |
| `pt-BR` | Portugiesisch (Brasilien) |

UE nutzt BCP-47-Codes. Region-Varianten (`de-AT`, `en-GB`) fallen automatisch auf den Basis-Code zurück.

## Text-Lokalisierung kombinieren

Der Text (`DialogueText`) auf jeder SayLine ist ein `FText` und durchläuft UEs Standard-Lokalisierungs-Pipeline:

- Localization Dashboard sammelt alle Texte (Gather)
- Export als `.po`-Dateien für Übersetzer
- Import übersetzter `.po`-Dateien
- Runtime-Culture-Switch übersetzt den Text sofort

Kombiniert mit der `DialogueVoice`-Map hast du pro Sprache sowohl den richtigen Text als auch die richtige Stimme.

## Culture im Preview-Runner wechseln

Der [Preview-Runner](../editor/preview-runner.md) hat ein Culture-Dropdown. Ein Klick wechselt die aktive Culture – du hörst sofort das richtige Voice-Asset und siehst den übersetzten Text, ohne den Editor neu zu starten.

> 📸 **Bild-Platzhalter:** `localization-preview-runner-culture-dropdown.png` — Preview-Runner mit geöffnetem Culture-Dropdown, aktive Culture "de".
> *Setup:* Preview-Runner-Panel im Editor unten oder als eigenes Fenster. Oben im Panel ein Dropdown-Menü mit Culture-Auswahl. Dropdown offen, Optionen: en, de, ja, fr. "de" ist markiert. Darunter der aktive Dialog-Text auf Deutsch sichtbar.

## Babel als Fallback für fehlende Sprachen

Wenn eine SayLine kein Voice-Asset für die aktuelle Culture hat (und kein Default-Key vorhanden ist), greift Babel automatisch – sofern `bEnableBabelVoice` in den Project Settings aktiv ist. Babel synthetisiert aus dem Text-String, unabhängig von der Sprache.

{% hint style="info" %}
**Babel als Entwicklungs-Platzhalter:** In frühen Projektphasen reicht es, nur `"en"`-Voice-Assets zu setzen. Für alle anderen Kulturen klingt Babel. So kannst du Lokalisierungs-Flows testen, ohne alle Aufnahmen fertig zu haben.
{% endhint %}

## Hinweise zur Asset-Verwaltung

{% hint style="warning" %}
Voice-Assets werden als **Hard-Reference** gespeichert. Sie werden beim Kochen des Packages mit-gepackt. Bei großen Voice-Libraries: `Streaming`-Flag in den SoundWave-Properties aktivieren, damit der Memory-Footprint beim Level-Start kontrolliert bleibt.
{% endhint %}
