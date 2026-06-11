---
description: Der vollständige Lokalisierungs-Workflow – FText-Quelltext, der Loc-Audit-Hint, String Tables, das UE Localization Dashboard, Voice pro Culture und die Culture-Preview im Editor.
---

# Dialoge lokalisieren

## Szenario

Dein Spiel erscheint in mehreren Sprachen. Jede Dialog-Zeile braucht übersetzten Text und idealerweise eine passende Voice-Aufnahme pro Sprache. Dieses Rezept ist der **End-to-End-Lokalisierungs-Weg**: vom Anlegen gather-stabilen Quelltexts über die Übersetzung (Dashboard oder CSV) bis zu Voice pro Culture und einer Editor-Preview, die Text und Voice gemeinsam umschaltet.

Es verbindet drei Tools, die jeweils ihre eigene Seite haben – hier sind sie zu einem Workflow gereiht.

## Was du lernst

- Warum Dialog-Text `FText` sein muss (nicht `FString`) und wie Keys ihn gather-stabil machen.
- Den **Loc-Audit**-Validator-Hint lesen und auflösen.
- **Migrate Texts to String Table** für zuverlässiges Gathering ausführen.
- Das UE **Localization Dashboard** treiben: Gather → Übersetzen → Compile.
- **Voice pro Culture** über `VoicePerCulture` zuweisen.
- Eine Culture im Editor previewen – Text **und** Voice schalten gemeinsam um.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- Die gewünschten Cultures in **Project Settings → Internationalization** registriert (z.B. nativ `en`, plus `de`, `ja`).

## Schritt-für-Schritt

### 1. Text als FText anlegen

Jedes Dialog-Text-Feld – `DialogueText` an der SayLine, `PromptText` und `ChoiceText` an der PlayerChoice – ist bereits ein `FText`. Das ist die Voraussetzung für Lokalisierung: `FText` trägt einen Namespace/Key, und die UE-Gather-Pipeline sammelt nur `FText`, nie rohe Strings. Tippe deine Quell-Culture-Zeilen einfach normal ein; beim Anlegen musst du nichts Besonderes tun.

{% hint style="info" %}
**Sprecher-Anzeigenamen auch.** Halte den Sprecher-`DisplayName` im Speakers-Panel des Assets als `FText`, damit er neben den Zeilen gegathert wird. Ein `FString`-Anzeigename wird nicht übersetzt.
{% endhint %}

### 2. Den Loc-Audit-Hint lesen

Dialog-Asset öffnen und kompilieren. Der Validator führt einen **Loc-Audit**-Check aus: Für jeden nicht-leeren SayLine-/PlayerChoice-/Choice-Text, der nicht von einer String Table gestützt wird und keinen stabilen Lokalisierungs-Key hat, gibt er einen blauen **Hint** an der Node aus. Der Hint ist informativ – der Dialog kompiliert trotzdem – aber er markiert Text, der für einen lokalisierten Build evtl. nicht zuverlässig gegathert wird.

Der Hint fasst die betroffenen Felder zusammen und weist auf den Fix hin: **Migrate Texts to String Table** ausführen oder dem Text anderweitig einen stabilen Key geben.

> 📸 **Bild-Platzhalter:** `localization-loc-audit-hint.png` — Eine SayLine-Node mit dem blauen Loc-Audit-Hint im Validierungs-Panel.
> *Setup:* Dialog-Asset offen. Eine SayLine-Node im Graph. Das Validierungs-/Message-Panel (unten) zeigt einen blauen Hint-Eintrag: "Loc-Audit: 'DialogueText' hat keinen stabilen Lokalisierungs-Key – führe Migrate Texts to String Table aus". Die Node selbst hat einen gedämpften blauen Indikator (kein roter Fehler-Rahmen). Roter Pfeil auf die Hint-Zeile.

### 3. Zu einer String Table migrieren

Öffne das Asset und nutze die Menüleiste des Editors: **Localization → Migrate Texts to String Table** (es liegt im Editor-Menü, nicht in der Toolbar, und gilt nur für ein einzelnes Asset). Das Plugin erstellt/aktualisiert eine `UStringTable` neben dem Asset (Suffix `_Texts`) und bindet jedes Text-Feld an einen String-Table-Eintrag mit stabilem Key (`NodeGuid.FieldId`). Der Loc-Audit-Hint verschwindet. Jetzt ist der Text Engine-nativ und gathert zuverlässig.

> 📸 **Bild-Platzhalter:** `localization-migrate-string-table.png` — Die Menü-Aktion und das resultierende `_Texts`-String-Table-Asset.
> *Setup:* Split-Ansicht. Links: Asset-Editor-Menüleiste offen beim **Localization**-Abschnitt mit hervorgehobener **Migrate Texts to String Table**-Aktion (roter Pfeil). Rechts: Content-Browser mit `DA_Villager_Intro` neben dem generierten `DA_Villager_Intro_Texts`-String-Table-Asset.

{% hint style="info" %}
**Zwei gültige Wege.** Die Migration zu einer String Table ist der gather-stabile, Dashboard-freundliche Weg. Nutzt du stattdessen die [CSV-Pipeline](../audio/csv-pipeline.md), macht deren Translation-Table die Übersetzungen ebenfalls ohne Dashboard live. Wähle, was zu deinem Team passt – sie können koexistieren.
{% endhint %}

### 4. Gather, Übersetzen, Compile (Localization Dashboard)

**Window → Localization Dashboard.** Wähle das Lokalisierungs-Target deines Projekts (den Eintrag, der den gegatherten Text des Plugins enthält – standardmäßig das **Game**-Target, das `Content/` inklusive deiner Dialog-String-Tables gathert).

1. **Gather Text** – sammelt alle `FText`-/String-Table-Einträge, inklusive deines migrierten Dialog-Texts.
2. **Die Target-Cultures hinzufügen** (`de`, `ja`, …), falls nicht vorhanden.
3. **Übersetzungen editieren** – den Culture-Editor öffnen und die übersetzten Strings eingeben.
4. **Compile Text** – schreibt die `.locres`-Dateien, die die Runtime liest.

> 📸 **Bild-Platzhalter:** `localization-dashboard-gather.png` — Localization Dashboard nach erfolgreichem Gather + Compile.
> *Setup:* Window → Localization Dashboard offen. Die Game-Target-Zeile mit grünem Status sichtbar. Culture-Liste mit en/de/ja und Übersetzungs-Fortschrittsbalken. Die Buttons Gather, Compile und Edit sichtbar. Roter Pfeil auf den Compile-Button.

### 5. Voice pro Culture zuweisen

Übersetzter Text ist die halbe Miete; für vertonte Spiele weist du pro Sprache eine Aufnahme zu. Fülle an jeder SayLine die `VoicePerCulture`-Map: Key = BCP-47-Culture-Code (`en`, `de`, `ja`), Value = das `USoundBase` für diese Sprache. Der Default-`DialogueVoice`-Slot ist der Fallback, wenn kein Culture-Key matcht. Volle Details: [Lokalisierung (VoicePerCulture)](../audio/localization.md).

> 📸 **Bild-Platzhalter:** `localization-voiceperculture-recipe.png` — SayLine-Details mit der für drei Cultures gefüllten VoicePerCulture-Map.
> *Setup:* SayLine-Node ausgewählt. Details "SayLine → Audio": `DialogueVoice` = `SW_Villager_Line01` (Fallback), und die `VoicePerCulture`-Map mit `en → SW_..._EN`, `de → SW_..._DE`, `ja → SW_..._JA`. Roter Pfeil auf die Map.

### 6. Eine Culture im Editor previewen

Öffne den [Preview-Runner](../editor/preview-runner.md) und wechsle das Culture-Dropdown. **Text und Voice schalten gemeinsam um**: Du siehst sofort die übersetzte Zeile und hörst die passende `VoicePerCulture`-Aufnahme, ohne den Editor neu zu starten. Das ist die schnelle Schleife, um zu prüfen, ob eine Übersetzung in der richtigen Länge gut liest und ob der richtige Take spielt.

> 📸 **Bild-Platzhalter:** `localization-preview-culture-switch.png` — Preview-Runner mit Culture-Dropdown auf `de`, zeigt die deutsche Zeile.
> *Setup:* Preview-Runner-Panel. Culture-Dropdown oben auf "de" (offen oder mit Auswahl). Die aktive Zeile auf Deutsch angezeigt. Ein kleines Lautsprecher-Icon zeigt an, dass der deutsche Voice-Take spielt. Caption: "Text + Voice schalten gemeinsam um."

## Der Workflow im Überblick

```text
1. FText-Quell-Zeilen anlegen        (einfach tippen)
2. Compile → Loc-Audit-Hint lesen    (markiert gather-instabilen Text)
3. Migrate Texts to String Table     (gather-stabile Keys)
4. Localization Dashboard             (Gather → Übersetzen → Compile)
5. VoicePerCulture pro SayLine füllen (Audio pro Sprache)
6. Preview-Runner-Culture-Dropdown    (Text + Voice schalten gemeinsam)
```

## Variation / Weiter gehen

- **CSV statt (oder neben) dem Dashboard**: das ganze Text-Set einem Übersetzer als Tabelle übergeben – siehe [CSV-Writer-Pipeline](../audio/csv-pipeline.md). Die importierte Translation-Table klingt ohne jeden Dashboard-Schritt.
- **Teilvertonung**: nur die `en`-Voice füllen, andere leer lassen; der Babel-Synth (falls aktiv) füllt fehlende-Sprache-Audio, sodass du ship-testen kannst, bevor alle Takes existieren.
- **Culture zur Laufzeit wechseln**: `Set Current Culture` (Blueprint) oder `FInternationalization::Get().SetCurrentCulture(...)` (C++); der nächste Dialog-Start nutzt die neue Culture für Text und Voice.
- **CJK-/RTL-Schriften**: ein Übersetzungs-Problem nur, wenn der UMG-Font die Glyphen nicht hat – gib dem Dialog-Widget einen Font mit der nötigen Schrift oder eine Composite-Font-Fallback-Kette.

## Troubleshooting

**Text bleibt nach dem Culture-Wechsel in der Quellsprache.**
Das Lokalisierungs-Target wurde nach dem Gather nicht **kompiliert**, also existiert keine `.locres`. Compile Text erneut ausführen. Außerdem prüfen, dass der Dialog-Text zu einer String Table migriert wurde oder anderweitig einen stabilen Key hat – ein gather-instabiles Literal (vom Loc-Audit markiert) ist evtl. gar nicht in der `.locres`.

**Loc-Audit-Hint verschwindet nicht.**
Das Text-Feld hat weiterhin keinen stabilen Key. Führe **Migrate Texts to String Table** am Asset aus. Leere und Culture-invariante Texte werden vom Audit absichtlich übersprungen und zeigen nie einen Hint.

**Übersetzter Text erscheint, aber die alte Voice spielt.**
Die `VoicePerCulture`-Map hat keinen Eintrag für die aktive Culture, also fällt sie auf `DialogueVoice` zurück. Den Culture-Key hinzufügen oder den Fallback akzeptieren. Region-Varianten (`de-AT`) fallen automatisch auf den Basis-Code (`de`) zurück.

**Gather findet den Dialog-Text nicht.**
Der Text ist irgendwo ein roher `FString`, oder er wurde als Blueprint-Literal statt am Asset platziert. Nur `FText`-Felder im Dialog-Graph (und String Tables) werden gegathert. Zur Sicherheit zu einer String Table migrieren.

## Siehe auch

- [CSV-Writer-Pipeline](../audio/csv-pipeline.md) – Tabellen-Round-Trip für Lektorat und Übersetzung.
- [Lokalisierung (VoicePerCulture)](../audio/localization.md) – die Voice-Map pro Culture im Detail.
- [Mehrsprachigkeit](multilingual-dialogue.md) – ein Voice-Map-fokussiertes Begleit-Rezept.
- [Preview-Runner](../editor/preview-runner.md) – die Culture-umschaltende Preview.
