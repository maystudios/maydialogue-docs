# Einfaches NPC-Gespräch

Das absolute Minimum: Ein NPC begrüßt den Spieler, sagt zwei Zeilen und beendet das Gespräch. Kein Branching, keine Bedingungen, keine Effekte. Dieses Rezept schärft dein Gefühl dafür, wie SayLine und Advance-Mode zusammenspielen.

## Szenario

Ein Dorfbewohner im Startdorf gibt dem Spieler Begrüßung + Kontext, dann schließt der Dialog automatisch. Kein Quest-Handling, kein Zustand.

## Beteiligte Nodes

* 1 × [Entry](../nodes/core/entry.md)
* 2 × [SayLine](../nodes/core/say-line.md)
* 1 × [Exit](../nodes/core/exit.md)

## Graph-Mock-Up

```
[Entry]
   │
   ▼
[SayLine: Dorfbewohner
   "Willkommen in Kreuzhof, Fremder."
   AdvanceMode: Manual]
   │
   ▼
[SayLine: Dorfbewohner
   "Pass auf dich auf. Nachts gehen komische Dinge um."
   AdvanceMode: Manual]
   │
   ▼
[Exit: Completed]
```

## Schritt-für-Schritt

1. **Asset anlegen**: *Content Browser → Rechtsklick → Miscellaneous → May Dialogue Asset*. Nenne es `DA_Villager_Intro`.
2. **Speakers-Panel öffnen** (*Window → Speakers*). Neuer Eintrag:
   * Name: *Dorfbewohner*
   * SpeakerTag: `Dialogue.Speaker.Villager`
3. **Graph vorbereiten**: Der Entry-Node existiert schon. Zieh vom Entry-Output-Pin einen Draht nach rechts los und wähle *Create Node → Say Line*. Verbinde den Output mit einem weiteren Say-Line und dann mit einem *Exit*-Node.
4. **Erste SayLine** konfigurieren:
   * `SpeakerTag`: `Dialogue.Speaker.Villager`
   * `DialogueText`: *„Willkommen in Kreuzhof, Fremder."*
   * `AdvanceModeOverride`: `Manual`
5. **Zweite SayLine** analog mit dem zweiten Text.
6. **Exit-Node**: `ExitStatus = Completed`. (Default, musst du nicht ändern.)
7. **Compile** (Button oben im Asset-Editor). Wenn keine Fehler in der Outline erscheinen, bist du fertig.

## Advance-Mechanik, die du verstehen musst

`AdvanceMode = Manual` bedeutet: Die Instance geht in `WaitingForAdvance` und verharrt dort, bis **jemand** `AdvanceDialogue()` aufruft. In der Standard-UI übernimmt das der Klick/Tastendruck auf die Textbox. Die drei wichtigsten Modi:

| Mode | Advance durch |
| --- | --- |
| `Manual` | Spieler-Input (Klick / Enter) |
| `Timer` | `AutoAdvanceDelay` verstreicht |
| `AfterVoice` | Voice-Asset-Wiedergabe endet; Fallback Typewriter-Ende |

Für Tutorials oder Cutscenes nimmst du meistens `Timer` oder `AfterVoice`, für interaktive NPC-Gespräche `Manual`.

## Runtime-Trigger

Im einfachsten Fall reicht ein Blueprint-Call, wenn der Spieler den NPC interagiert:

```
[Event: OnInteract]
  │
  ▼
[MayDialogueLibrary :: Start Dialogue]
  ├ WorldContext: Self
  ├ Asset:        DA_Villager_Intro
  ├ Instigator:   Player Pawn
  └ Target:       Self (der NPC)
```

In C++ äquivalent:

```cpp
if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
{
    Sub->StartDialogue(Asset, PC->GetPawn(), this);
}
```

## Testen im Preview-Runner

Du musst das nicht in PIE testen. Öffne das Asset, klicke *Play* oben rechts (der [Preview Runner](../editor/preview-runner.md)) – das Gespräch läuft direkt im Editor-Fenster mit dem Slate-Debug-Widget. Advance mit *Space*, Abbruch mit *Esc*.

## Troubleshooting

### Der Dialog wird sofort nach der ersten Zeile beendet

* **Ursache 1**: Der Output-Pin der ersten SayLine zeigt versehentlich direkt auf den Exit statt auf die zweite SayLine. Prüfe die Verbindung.
* **Ursache 2**: Du hast `AdvanceMode = Immediate` gesetzt. Das zippt durch alle Nodes ohne Pause. Wechsle auf `Manual`.

### Der Spieler-Input löst den Advance nicht aus

* Prüfe in den [Projekt-Einstellungen](../getting-started/project-settings.md), ob `bSwitchToUIInputDuringDialogue = true` gesetzt ist. Sonst kommt der Klick nicht bei der Widget-Layer an.
* Das Widget braucht *Focus*. Wenn du ein eigenes UMG-Widget hast, stelle sicher, dass `SetKeyboardFocus` bei `NativeConstruct` aufgerufen wird.

### Der Name „Dorfbewohner" erscheint als leere Zeile

* Im Speakers-Panel ist kein `DisplayName` gesetzt – nur der Tag. Fülle das Feld `DisplayName` aus oder überschreibe den Namen direkt am Speaker im Dialog-Asset.

## Nächster Schritt

Sobald dieses Minimum läuft, kannst du jederzeit zu den spannenderen Rezepten springen:

* [Zufällige Begrüßungen](random-greetings.md) – ersetze die erste SayLine durch eine RandomLine, damit der NPC nicht immer dasselbe sagt.
* [Verzweigungen mit Bedingungen](branching-conditions.md) – füge einen PlayerChoice hinzu.

{% hint style="info" %}
Wenn du die beiden SayLines später als Gemeinsam-Block an mehreren Stellen brauchst, lagere sie in ein eigenes Asset aus und referenziere sie per [Link](../nodes/core/link.md). Siehe [Wiederverwendbare Dialog-Fragmente](linking-dialogues.md).
{% endhint %}
