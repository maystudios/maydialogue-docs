# Lokalisierung (VoicePerCulture)

MayDialogue unterstützt **pro-Kultur Voice-Assets**. Zur Laufzeit wählt das System basierend auf der aktiven UE-Culture das passende Asset.

## Datenmodell

Auf jeder SayLine:

```cpp
UPROPERTY(EditDefaultsOnly, Category = "Audio")
TMap<FString, USoundBase*> DialogueVoice;
```

Key: Culture-Code (`"en"`, `"de"`, `"ja"`, `"fr"` …). Value: das Voice-Asset für diese Culture.

## Auflösungs-Logik

1. System fragt UE-Locale-API nach der aktuellen Culture.
2. Lookup in `DialogueVoice[Culture]`.
3. Wenn nicht vorhanden: Fallback auf den **Default-Key** (typisch `"en"` oder leerer String).
4. Wenn auch das fehlt: keine Voice → Babel oder Silence.

## FText für Text

Alle Text-Properties sind `FText`, d.h. sie gehen automatisch in UE's Lokalisierungs-Flow:

* Text-Gather-Pass (`Localization Dashboard`).
* Export/Import als PO-Files.
* Runtime-Cultures switch übersetzt Text sofort.

Kombiniert mit `DialogueVoice` hast du für jede Sprache:

* Text (übersetzt via FText).
* Voice (aus dem Map-Key).

## Beispiel-Konfiguration

SayLine „Hallo"  
DialogueText (FText):

* Namespace: `Dialog_Greeting`
* Key: `hello_line`
* Default-String: `"Hallo."` (wird im Gather gesammelt und in `.po`-File angepasst).

DialogueVoice (Map):

* `"en"` → `VO_EN_Hello`
* `"de"` → `VO_DE_Hallo`
* `"ja"` → `VO_JA_Konnichiwa`

Wenn die Culture `de-DE` ist:

* Text: *„Hallo."* (aus PO-File, sofern importiert).
* Voice: `VO_DE_Hallo`.

## Culture-Switch im Preview-Runner

Der [Preview-Runner](../editor/preview-runner.md) hat ein **Culture-Dropdown**. Wechsel wirkt sofort – du testest Lokalisierung ohne Engine-Neustart.

## Babel und Lokalisierung

Wenn eine SayLine **kein** Voice-Asset für die aktuelle Culture hat:

1. System schaut nach Default-Key.
2. Wenn auch nicht: Babel-Synthese läuft am Text (der ja bereits lokalisiert ist).

Babel funktioniert **kulturagnostisch** – es synthetisiert aus dem Text-String, unabhängig von der Sprache.

## Locale-Code-Konventionen

UE nutzt BCP-47-Codes. Übliche Keys:

| Code | Sprache |
| --- | --- |
| `en` | Englisch |
| `de` | Deutsch |
| `fr` | Französisch |
| `ja` | Japanisch |
| `zh-Hans` | Chinesisch (vereinfacht) |
| `es-419` | Spanisch (Lateinamerika) |

Die Abgleichs-Logik findet auch Regions-Varianten:

* Gesucht: `de-AT` → Fallback: `de` → Fallback: Default-Key.

## Anmerkungen

* Voice-Assets als **Hard-Reference** (nicht SoftObjectPtr) – damit sie beim Package-Cooker mit-gepackt werden.
* Große Voice-Libraries: mit **Streaming-SoundWaves** kombinieren, damit Memory beim Level-Start nicht explodiert.
