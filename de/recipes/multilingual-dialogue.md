---
description: Text per FText-Gather lokalisieren und Voice-Files pro Kultur zuweisen – der komplette L10n-Workflow.
---

# Mehrsprachigkeit

## Szenario

Dein Spiel erscheint auf Deutsch, Englisch und Japanisch. Ein NPC sagt zwei Zeilen, jede mit eigener Vertonung pro Sprache. Dieses Rezept zeigt den kompletten Weg: Kulturen konfigurieren, Text gathern, Voice-Map befüllen, Sprache zur Laufzeit wechseln.

## Was du lernst

- Kulturen im UE-Projekt registrieren.
- FText-Gather für Dialog-Assets ausführen.
- `VoicePerCulture`-Map am SayLine-Node befüllen.
- Laufzeit-Sprachwechsel per Blueprint und C++.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- UE-Localization-Target für das Projekt angelegt.
- Voice-Files als `USoundBase` importiert: `SW_Villager_Line01_DE`, `_EN`, `_JA`.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Dorfbewohner – "Guten Tag, Fremder."  VoicePerCulture: {de, en, ja}]
   │
   ▼
[SayLine: Dorfbewohner – "Seid auf der Hut."    VoicePerCulture: {de, en, ja}]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `multilingual-dialogue-graph-overview.png` — Asset-Editor mit einer SayLine, VoicePerCulture-Map ausgeklappt.
> *Setup:* Asset `DA_Villager_Intro` geöffnet. SayLine ausgewählt, Details-Panel rechts zeigt `DialogueText = "Guten Tag, Fremder."` und darunter `VoicePerCulture`-Map mit drei Zeilen: `de → SW_Villager_Line01_DE`, `en → SW_Villager_Line01_EN`, `ja → SW_Villager_Line01_JA`.

## Schritt-für-Schritt

### 1. Kulturen im Projekt registrieren

**Edit → Project Settings → Internationalization**:
- `Native Culture`: `de`
- `Cultures to Stage`: `de`, `en`, `ja`

### 2. Dialog-Asset anlegen und Text eingeben

Asset: `DA_Villager_Intro`. SayLine `DialogueText` in der Native-Culture (Deutsch): *„Guten Tag, Fremder."*

`FText` speichert beim Asset-Save automatisch einen Text Key – das Gather-System nutzt diesen später.

### 3. Voice-per-Culture-Map befüllen

Am SayLine-Node unter `DialogueVoice` → Map mit drei Einträgen:

| Key | Value |
|-----|-------|
| `de` | `SW_Villager_Line01_DE` |
| `en` | `SW_Villager_Line01_EN` |
| `ja` | `SW_Villager_Line01_JA` |

Für den Fallback-Key `""` (leerer String): wird genutzt, wenn die aktuelle Kultur nicht in der Map ist. Leer lassen wenn Babel-Synthese aktiv sein soll.

### 4. Text-Gather ausführen

**Window → Localization Dashboard** → Gather Text → Compile Text → Edit (für Übersetzungseingabe).

Per CLI:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Binaries/Win64/UnrealEditor-Cmd.exe" ^
  "C:/UnrealEngine/VHS/VHS.uproject" ^
  -run=GatherText -Target=Game
```

> 📸 **Bild-Platzhalter:** `multilingual-dialogue-localization-dashboard.png` — Localization Dashboard nach erfolgreichem Gather.
> *Setup:* Window → Localization Dashboard geöffnet. Game-Target sichtbar, Status-Spalte zeigt grünen Haken nach Gather+Compile. Kulturen-Liste mit de/en/ja und Fortschritts-Balken.

### 5. Übersetzungen eintragen

Im Culture-Editor (Compile → Edit) die englischen und japanischen Texte eingeben. MayDialogue nutzt beim Dialog-Start die aktive UE-Kultur für die Textanzeige.

### 6. Laufzeit-Sprachwechsel

**Blueprint:**

```text
[Set Current Culture]
   └─ Culture: "en"
```

**C++:**

```cpp
FInternationalization::Get().SetCurrentCulture(TEXT("en"));
```

Beim nächsten Dialog-Start nutzt das Plugin die neue Kultur für Text und Voice-Auswahl.

## Voice-Auflösung

```text
ExecuteNode SayLine
   → Aktuelle Kultur holen (FInternationalization)
   → Key in VoicePerCulture?
       ├─ Ja → Voice abspielen
       └─ Nein → Fallback-Key ("") gesetzt?
                   ├─ Ja → Fallback-Voice abspielen
                   └─ Nein → Babel aktiv? → Synthese / Stille
```

## Empfohlene Ordner-Struktur

```text
Content/
  Dialogue/
    Villager/
      DA_Villager_Intro.uasset
      Voice/
        de/SW_Villager_Line01.wav
        en/SW_Villager_Line01.wav
        ja/SW_Villager_Line01.wav
```

Über Kultur-Ordner kannst du in Perforce gezielt Culture-Sets beim Packaging ein- oder ausschließen.

## Mix: Nur teilweise vertont

| Szenario | Setup |
|----------|-------|
| Nur EN vertont, andere nur Text | Einzigen `en`-Key füllen |
| DE + EN vertont, JA nur Text | `de` + `en` befüllen, `ja` leer lassen |
| Alle Kulturen über Babel | VoiceMap leer, Babel in Projekt-Settings aktivieren |

> 📸 **Bild-Platzhalter:** `multilingual-dialogue-ingame-preview.png` — Preview-Runner mit aktivem Dialog auf Englisch.
> *Setup:* Preview-Runner Panel mit englischer Kultur. Textbox zeigt `"Good day, stranger."`, Speaker-Name `Villager`. Hinweis: Culture muss vor dem Preview-Start gesetzt sein.

## Variation / Weiter gehen

- Babel-Synthese für fehlende Kulturen aktivieren → [Audio → Babel-System](../audio/babel-system.md).
- Speaker-DisplayName ebenfalls als FText pflegen (nicht als FString), damit er mitgegathert wird.
- Kultur-spezifische Emotionen: EmotionTags sind GameplayTags (nicht lokalisierbar), also sprachunabhängig.

## Troubleshooting

**Englische Voice spielt, Text bleibt deutsch.**
Localization-Target nicht Compiled nach dem Gather. `.locres`-Datei fehlt. Re-Run Compile Text.

**Voice-Fallback greift nicht.**
Leerer `""`-Key nicht befüllt. Entweder Babel aktivieren (`bEnableBabelVoice` in Projekt-Settings) oder den Key explizit füllen.

**Gather findet den Text nicht.**
Text wurde per Blueprint-Literal in ein Variable-Feld gelegt statt direkt am Asset. Nur `FText`-Properties in Dialog-Graphs werden gegathert.

**Japanische Schrift nicht sichtbar.**
Das ist kein MayDialogue-Problem – das UMG-Widget braucht einen Font mit CJK-Support oder eine Font-Fallback-Kette (UE Composite Font Assets).
