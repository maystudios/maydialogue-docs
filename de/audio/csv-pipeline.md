---
description: Exportiere deinen Dialog-Text als CSV, lass ihn lektorieren oder übersetzen und importiere ihn zurück – der Round-Trip-Workflow für Lektorat und Lokalisierung.
---

# CSV-Writer-Pipeline: Export, Lektorat, Import

## Wann brauche ich das?

Wenn der Text in deinen Dialog-Assets für eine Weile den Editor verlassen muss:

- Ein **Lektor** will Tippfehler korrigieren und die Prosa straffen – in einer Tabelle, nicht im Graph-Editor.
- Eine **Übersetzungsagentur** braucht jede Zeile in einer einzigen Datei, eine Spalte pro Sprache, und schickt sie ausgefüllt zurück.
- Du willst einen **Review-Snapshot** aller Dialog-Texte an einem Ort – für ein Skript-Read-Through oder eine Wortzählung.

Die CSV-Pipeline schickt den Text eines Dialog-Assets durch eine einfache Tabellen-Datei und wieder zurück. Der Export schreibt eine Zeile pro Text-Feld; du (oder ein externer Lektor) änderst die Zellen; der Import schreibt den lektorierten Quelltext zurück in den Graph und registriert die übersetzten Spalten, sodass sie zur Laufzeit klingen – **ohne das Localization Dashboard anzufassen.**

Das ist ein **Zero-Code**-Workflow. Alles läuft über Toolbar-Buttons und Rechtsklick-Aktionen.

## Das große Bild

```text
[Dialog-Asset]
     │  Export Texts (CSV)
     ▼
[texts.csv]  ── an Lektor / Übersetzer schicken ──▶  [texts_filled.csv]
                                                            │  Import Texts (CSV)
                                                            ▼
            ┌──────────────────────────────────────────────────────────┐
            │ SourceText-Spalte  → zurück in den Graph geschrieben       │
            │ Culture-Spalten     → UMayDialogueTranslationTable-Asset    │
            │ (Auto-Recompile + dirty)                                   │
            └──────────────────────────────────────────────────────────┘
```

> 📸 **Bild-Platzhalter:** `csv-pipeline-overview-diagram.png` — Pfeil-Diagramm des Export → Edit → Import Round-Trips.
> *Setup:* Eine Grafik erstellen (kein Editor-Screenshot). Linke Box `[Dialog-Asset]` → Pfeil "Export Texts (CSV)" → Box `[texts.csv]` → Pfeil "Lektorat / Übersetzung" → Box `[texts_filled.csv]` → Pfeil "Import Texts (CSV)" → aufgeteilt in zwei Boxen `[SourceText → Graph]` und `[Culture-Spalten → Translation Table]`. Horizontales Layout, weißer Hintergrund, dunkelgraue Pfeile.

## Zero-Code-Weg

### 1. Texte exportieren

Öffne das Dialog-Asset. In der Asset-Editor-Toolbar auf **Export Texts** klicken (der Button heißt "Export Texts"; sein Tooltip erwähnt CSV). Speicherort wählen (z.B. `DA_Villager_Intro_texts.csv`). Das Plugin schreibt eine Zeile pro lokalisierbarem Text-Feld.

> 📸 **Bild-Platzhalter:** `csv-pipeline-toolbar-export.png` — Asset-Editor-Toolbar mit hervorgehobenem Export-Texts-Button.
> *Setup:* Dialog-Asset im Graph-Editor geöffnet. Obere Toolbar sichtbar. Roter Pfeil auf den **Export Texts**-Button (Export-Icon). Daneben der **Import Texts**-Button (Import-Icon). Schwebender Tooltip: "Exportiert allen lokalisierbaren Text (Zeilen, Prompts, Choices) in eine CSV-Datei für die Übersetzung."

### 2. In einer Tabelle bearbeiten

Öffne die CSV in Excel, LibreOffice Calc oder Google Sheets. Die Spalten sind:

| Spalte | Bedeutung |
|---|---|
| `NodeGuid` | Stabile Adresse der Node. **Nicht ändern.** |
| `NodeType` | `SayLine`, `PlayerChoice`, … (Kontext, read-only). |
| `Field` | Welches Text-Feld der Node – `DialogueText`, `PromptText`, `Choice[2].ChoiceText`, … **Nicht ändern.** |
| `SpeakerTag` | Wer die Zeile spricht (nur Kontext). |
| `SourceText` | Die Zeile in der Quell-Culture – **hier fürs Lektorat editieren.** |
| `en`, `de`, `ja`, … | Eine Spalte pro Culture – **hier für die Übersetzung ausfüllen.** |

Editiere die `SourceText`-Zellen fürs Lektorat; fülle die Culture-Spalten für die Übersetzung. Lass `NodeGuid` und `Field` unangetastet – darüber findet der Import die richtige Node.

> 📸 **Bild-Platzhalter:** `csv-pipeline-spreadsheet.png` — Die exportierte CSV in einer Tabelle geöffnet, Culture-Spalten ausgefüllt.
> *Setup:* LibreOffice Calc (oder Excel) zeigt ein Sheet mit den Spalten NodeGuid, NodeType, Field, SpeakerTag, SourceText, en, de, ja. Drei oder vier Dialog-Zeilen. Die `de`- und `ja`-Spalten teilweise mit Übersetzungen gefüllt. Die NodeGuid-Spalte zeigt GUID-Strings (ausgegraut/schmal). Die SourceText- und Culture-Spalten als editierbar hervorheben.

### 3. Texte zurück-importieren

Zurück im Asset-Editor auf **Import Texts** klicken und deine bearbeitete Datei wählen. Das Plugin:

1. Schreibt die `SourceText`-Spalte zurück in die Text-Felder des Graphs (gematcht über `NodeGuid` + `Field`).
2. Sammelt die nicht-leeren Culture-Zellen in ein **`UMayDialogueTranslationTable`**-Asset neben deinem Dialog (Suffix `_Translations`).
3. Rekompiliert das Asset und markiert es dirty.

Asset speichern. Fertig – der lektorierte Quelltext steht im Graph, die Übersetzungen sind live.

> 📸 **Bild-Platzhalter:** `csv-pipeline-import-result.png` — Asset-Editor nach dem Import, mit der generierten Translation-Table im Content-Browser sichtbar.
> *Setup:* Split-Ansicht. Links: Graph-Editor mit ausgewählter SayLine, deren `DialogueText` jetzt die korrigierte Quell-Zeile zeigt. Rechts: Content-Browser-Ordner mit dem Dialog-Asset `DA_Villager_Intro` neben dem auto-generierten `DA_Villager_Intro_Translations`-Data-Asset. Roter Pfeil auf die neue Translation-Table.

### 4. (Optional) Viele Assets auf einmal

Im **Content-Browser** ein oder mehrere Dialog-Assets auswählen, Rechtsklick, und die Batch-Aktionen **Localization → Export Texts (CSV)** / **Import Texts (CSV)** nutzen. Das exportiert/importiert, ohne jeden Asset-Editor zu öffnen – ideal, um einen ganzen `Content/Dialogue/`-Ordner in einem Rutsch an einen Übersetzer zu übergeben. (Der Batch-Export schreibt pro Asset eine `<AssetName>_Texts.csv` in einen von dir gewählten Ordner.)

{% hint style="info" %}
Das Content-Browser-Batch-Menü bietet nur **Export** und **Import**. **Migrate Texts to String Table** ist eine Einzel-Asset-Aktion – öffne das Asset und nutze die Menüleiste des Editors: **Localization → Migrate Texts to String Table** (siehe nächster Abschnitt).
{% endhint %}

> 📸 **Bild-Platzhalter:** `csv-pipeline-content-browser-context.png` — Content-Browser-Rechtsklick-Menü auf ausgewählten Dialog-Assets.
> *Setup:* Content-Browser mit drei ausgewählten (hervorgehobenen) `DA_*`-Dialog-Assets. Rechtsklick-Kontextmenü offen, mit aufgeklapptem **Localization**-Untermenü und den Einträgen **Export Texts (CSV)** und **Import Texts (CSV)**.

## Wie Übersetzungen ohne Dashboard live gehen

Die importierten Culture-Spalten werden als `FPolyglotTextData`-Einträge in einer `UMayDialogueTranslationTable` gespeichert. Dieses Asset **registriert sich beim Laden automatisch** (in `PostLoad`): es schiebt jeden Eintrag in den Text-Lokalisierungs-Manager der Engine, gekeyt über Namespace/Key des Text-Felds. Jedes `FText`, das diesen Key teilt – also deine Dialog-Zeilen – löst dann auf die passende Per-Culture-Zeichenkette für die aktive Culture auf.

Für Einsteiger heißt das: **du musst das Localization Dashboard gar nicht öffnen.** Wechsle die Spiel-Culture, und die importierten Übersetzungen erscheinen.

{% hint style="info" %}
**Das ist eine Komfort-Schicht, kein Ersatz.** Wenn du das UE Localization Dashboard (`.po`/`.locres`-Workflow) bereits nutzt, läuft es unverändert weiter – beide speisen denselben Text-Lokalisierungs-Manager. Den vollen Dashboard-Weg zeigt [Rezepte → Dialoge lokalisieren](../recipes/localization.md).
{% endhint %}

## Zu einer String-Table migrieren

Für ein voll gather-stabiles, Engine-natives Setup öffne das Asset und nutze die Menüleiste des Editors: **Localization → Migrate Texts to String Table**. Es erstellt/aktualisiert eine Per-Asset-`UStringTable` (Suffix `_Texts`) und bindet jedes Text-Feld an einen String-Table-Eintrag mit stabilem Key (`NodeGuid.FieldId`). Nach der Migration wird der Dialog-Text von der String-Table getrieben, sodass das UE Localization Dashboard ihn zuverlässig gathert.

Nimm das, wenn du dich langfristig auf den Dashboard-Workflow festlegst; nimm die einfache Translation-Table, wenn CSV-gelieferte Übersetzungen ohne weiteres Setup klingen sollen.

## API-Referenz

Die Editor-seitige Pipeline ist `FMayDialogueTextPipeline` (Modul `MayDialogueEditor`). Alle Entry-Points sind statisch; die Pipeline hält keinen State.

| Funktion | Zweck |
|---|---|
| `ExportCSV(Asset, Path, Cultures)` | Schreibt die Text-Felder des Assets in eine CSV-Datei. Eine Zeile pro Feld, eine Spalte pro Culture. |
| `ImportCSV(Asset, Path)` | Lädt eine CSV und importiert sie: Quelltext → Graph, Culture-Zellen → Translation-Table, dann Recompile + dirty. |
| `BuildCSV(Asset, Cultures)` | Baut den CSV-String im Speicher (von Export und Tests genutzt). |
| `ImportCSVString(Asset, CSVContent)` | Importiert einen In-Memory-CSV-String. Gibt true zurück, wenn mindestens eine Zeile eine Node gematcht hat. |
| `MigrateToStringTable(Asset)` | Erstellt/aktualisiert die `_Texts`-`UStringTable` und bindet Felder an stabile Keys. Gibt die Table zurück. |
| `DiscoverCultures(Asset)` | Berechnet das Culture-Spalten-Set (Projekt-Loc-Target-Cultures ∪ `VoicePerCulture`-Keys). |
| `CollectTextFields(Node, OutFields)` | Zählt die lokalisierbaren Felder einer Node auf (`DialogueText`, `PromptText`, `Choice[i].*`). |
| `MakeStableKey(NodeGuid, FieldId)` | Stabiler Key für ein Feld: `"<NodeGuid>.<FieldId>"`. |
| `MakeNamespace(Asset)` | Generierter Key-Namespace: `"MayDialogue.<AssetName>"`. |

**CSV-Format:** RFC-4180-konform – Felder werden gequotet, wenn sie Kommas, Anführungszeichen oder Zeilenumbrüche enthalten; innere Anführungszeichen werden verdoppelt. Spalten-Reihenfolge: `NodeGuid, NodeType, Field, SpeakerTag, SourceText, <culture…>`.

**Runtime-Asset:** `UMayDialogueTranslationTable` (`Entries: TArray<FPolyglotTextData>`) registriert via `RegisterAll()` aus `PostLoad()`. Es ist `BlueprintType`; `RegisterAll` ist `BlueprintCallable` für Projekte, die die Table zur Laufzeit aufbauen.

## Edge-Cases & Stolpersteine

{% hint style="warning" %}
**Excel und das Semikolon-Trennzeichen.** Die Pipeline schreibt **Komma-getrennte UTF-8 mit BOM**. Die BOM sorgt dafür, dass modernes Excel die UTF-8-Kodierung korrekt erkennt – Umlaute und nicht-lateinische Schriften überstehen einen einfachen Doppelklick. Die verbleibende Falle ist das **Trennzeichen**: Auf einem deutschen/europäischen Locale erwartet Excel evtl. weiterhin ein **Semikolon** statt eines Kommas und kippt jede Zeile in eine einzige Spalte. Falls das passiert, nutze **Daten → Aus Text/CSV** und wähle **Komma** explizit als Trennzeichen. LibreOffice Calc zeigt den Import-Dialog automatisch – dort **UTF-8** belassen und **Komma** wählen. Beim Zurückspeichern "CSV UTF-8" wählen und das Komma-Trennzeichen behalten.
{% endhint %}

**Von Hand editierter Graph-Text wird beim Import überschrieben.** Die `SourceText`-Spalte ist maßgeblich: Der Import schreibt sie zurück in die Graph-Node. Wenn du eine Zeile direkt im Graph editiert hast, *nachdem* du exportiert hast, geht diese Änderung verloren, sobald du die ältere CSV importierst. Re-exportiere vor dem Import, falls sich der Graph zwischenzeitlich geändert hat.

**Niemals die Spalten `NodeGuid` und `Field` umsortieren oder umbenennen.** Sie bilden die Adresse, auf die der Import matcht. Zeilen, deren `NodeGuid` keine Node mehr auflöst (z.B. weil die Node gelöscht wurde), werden einfach übersprungen. `NodeType` und `SpeakerTag` sind nur Kontext – der Import ignoriert sie.

**Der Import rekompiliert das Asset.** Da der Import-Pfad dem etablierten Compile-on-Edit-Flow folgt, rekompiliert das Asset und wird direkt nach dem Import dirty markiert. Du musst das Asset (und das generierte Translation-/String-Table-Asset) trotzdem **speichern**, um die Änderung zu persistieren. Der Import schreibt in die **Source**-Graph-Node (aufgelöst über `FindSourceNodeByGuid`), sodass die Änderung den nächsten Recompile übersteht.

**Leere Culture-Zellen werden ignoriert.** Eine leere `de`-Zelle erzeugt keine (kaputte) leere deutsche Übersetzung – sie wird übersprungen, und die Zeile fällt nach der normalen `FText`-Auflösung zurück. Fülle nur die Zellen, die du wirklich hast.

**Eine Choice-Node erzeugt mehrere Zeilen.** Eine `PlayerChoice` exportiert ihren `PromptText` plus pro Choice eine `Choice[i].ChoiceText` (und `Choice[i].UnavailableReason`, wenn gesetzt). Halte sie zusammen – sie teilen dieselbe `NodeGuid`, unterscheiden sich aber im `Field`.

## Siehe auch

- [Dialoge lokalisieren](../recipes/localization.md) – das vollständige End-to-End-Lokalisierungs-Rezept (Dashboard + CSV + Voice pro Culture).
- [Lokalisierung (VoicePerCulture)](localization.md) – ein anderes Voice-Asset pro Culture.
- [Mehrsprachigkeit](../recipes/multilingual-dialogue.md) – der Voice-Map-fokussierte Walkthrough.
