---
description: Die Hauptbausteine des Plugins und wie sie zusammenarbeiten — aus Nutzersicht.
---

# Architektur im Großen

MayDialogue besteht aus vier Hauptbausteinen, die du als Nutzer kennen solltest: das **Dialog-Asset**, die **Participant-Komponente**, das **Subsystem** und die **UI-Schicht**. Dieses Kapitel zeigt, wie sie zusammenhängen — ohne in interne Klassen-Details zu gehen.

> 📸 **Bild-Platzhalter:** `architecture-overview.png` — Übersichtsgrafik aller vier Hauptbausteine.
> *Setup:* Eine einfache Blockdiagramm-Darstellung mit vier Kästen: „Dialog-Asset (Disk)", „Subsystem (Welt)", „Participant-Komponente (Actor)", „UI-Widget". Pfeile: Asset → Subsystem (wird geladen von), Participant-Komponente → Subsystem (startet über), Subsystem → UI-Widget (sendet Events an). Kein Code, nur Boxen und Beschriftungen.

## Die vier Hauptbausteine

### 1. Das Dialog-Asset

Ein Dialog-Asset (`UMayDialogueAsset`) ist eine Datei in deinem Content-Browser. Es enthält:

- Den **Node-Graph** — der gesamte Dialogverlauf, visuell als verbundene Boxen.
- Die **Sprecher-Liste** — wer spricht, mit welchem Portrait, welcher Farbe, welchem Audio-Setup.
- Die **Variablen-Deklarationen** — alle Variablen, die im Gespräch existieren.

Das Asset ist reine **Struktur und Daten**. Es läuft nichts, wenn du es nur im Content-Browser hast. Erst wenn ein Dialog gestartet wird, entsteht aus dem Asset eine laufende Instanz.

> 📸 **Bild-Platzhalter:** `asset-content-browser.png` — Content-Browser mit einem geöffneten Dialog-Asset.
> *Setup:* Content-Browser zeigt ein Asset `DA_Guard_Greeting` mit MayDialogue-Icon. Daneben der Asset-Editor mit geöffnetem Graph, sichtbar: Entry-Node (grün), eine SayLine (rote Title-Bar für Wächter-Sprecher), ein PlayerChoice-Node mit zwei Choices, Exit-Node (rot). Speakers-Panel rechts oben zeigt einen Eintrag „Wächter" mit orangefarbigem Farb-Chip.

### 2. Die Participant-Komponente

`UMayDialogueParticipant` ist ein **ActorComponent**, den du an jeden Actor hängst, der in Dialogen vorkommt — NPCs, den Spieler-Pawn, Objekte mit Stimme.

Die Komponente hat zwei Hauptaufgaben:

- **Identität**: Sie trägt den `ParticipantTag` (z.B. `Dialogue.Speaker.Guard`). Darüber verknüpft das Plugin zur Laufzeit den lebendigen Actor im Level mit dem Sprecher-Eintrag im Asset.
- **Gedächtnis**: In `PersistentMemory` speicherst du Variablen, die Gespräche überdauern (z.B. ob der Spieler den NPC schon kennengelernt hat).

> 📸 **Bild-Platzhalter:** `participant-component-details.png` — Details-Panel der Participant-Komponente an einem NPC.
> *Setup:* Ein Guard-Actor im Level ausgewählt, Details-Panel zeigt die `MayDialogueParticipant`-Komponente. Sichtbare Felder: `ParticipantTag = Dialogue.Speaker.Guard`, `DisplayName = Wächter`, `DefaultDialogue = DA_Guard_Greeting` (Asset-Referenz), `bAutoFacePartner = true`. Portrait-Slot ist befüllt mit einem Texture2D-Verweis.

### 3. Das Subsystem

`UMayDialogueSubsystem` ist ein **World-Subsystem** — es existiert einmal pro Welt und lebt automatisch, du musst es nicht anlegen.

Es ist die einzige Instanz, die neue Gespräche starten und beenden darf. Alle drei Start-Pfade (direkt auf der Komponente, über die Blueprint-Library, oder per Code) laufen am Ende beim Subsystem zusammen.

Das Subsystem verwaltet die aktive Instanz und räumt sie nach dem Ende automatisch auf.

> 📸 **Bild-Platzhalter:** `blueprint-start-dialogue.png` — Blueprint-Graph eines Interaktions-Triggers, der einen Dialog startet.
> *Setup:* Ein einfacher BP-Graph mit: `Event BeginOverlap` → `Start Default Dialogue` (MayDialogueLibrary-Node). Node-Pins sichtbar: `Instigator = Get Player Pawn`, `Target = Self (GuardActor)`. Der Start-Node ist violett (Library-Funktion), der Output-Pin zeigt eine Verbindung zu einem `Print String`-Node mit Text „Dialog gestartet".

```cpp
// C++ — drei gleichwertige Wege, einen Dialog zu starten

// 1. Auf der Komponente (OOP-Stil)
Guard->FindComponentByClass<UMayDialogueParticipant>()->StartDefaultDialogue(Player);

// 2. Über die Library (Blueprint-nah)
UMayDialogueLibrary::StartDialogue(this, DA_Guard_Greeting, PlayerPawn, GuardActor);

// 3. Direkt am Subsystem
UMayDialogueSubsystem::Get(this)->StartDialogue(DA_Guard_Greeting, PlayerPawn, GuardActor);
```

### 4. Die UI-Schicht

Das Plugin liefert eine fertige UI mit. Du musst kein Widget von Grund auf bauen, um Dialoge anzuzeigen.

Die UI-Schicht hört auf Events der aktiven Instanz (`OnMessageReceived`, `OnChoicesPresented`) und rendert Text, Portraits, Choices und Typewriter-Effekte. Du kannst das mitgelieferte Widget direkt nutzen, anpassen oder durch ein eigenes ersetzen.

> 📸 **Bild-Platzhalter:** `ui-ingame.png` — Laufender Dialog im PIE-Modus.
> *Setup:* PIE-Ansicht mit aktivem Dialog. Unten im Bild das Dialog-Widget: Portrait des Wächters links, Name „Wächter" in orangefarbener Schrift, darunter der Dialog-Text mit laufendem Typewriter-Effekt (Cursor sichtbar). Keine Choices sichtbar (SayLine läuft gerade). Hintergrund: das Level.

## Datenfluss einer Sprechzeile

So läuft eine einzelne SayLine durch das System:

```text
Subsystem
  └─→ Instance::ContinueToNode(SayLine-Node)
        └─→ Node erzeugt FMayDialogueMessage (Text, Sprecher, Tags)
              └─→ Instance broadcastet OnMessageReceived
                    ├─→ UI-Widget rendert Text + Typewriter
                    ├─→ Audio-System spielt Voice-Asset
                    └─→ Dein Quest-System / Analytics reagiert (optional)
```

Die Events sind **Multicast-Delegates** — beliebig viele Systeme können gleichzeitig lauschen, ohne sich gegenseitig zu stören.

MayDialogue ist nach Installation ohne weitere Einrichtung einsatzbereit. GAS-Funktionen sind direkt verfügbar, wenn dein Projekt das Gameplay Ability System nutzt.

## Zusammenfassung

| Baustein | Lebt | Zuständig für |
| --- | --- | --- |
| Dialog-Asset | Auf Disk / im Content-Browser | Struktur: Nodes, Sprecher, Variablen |
| Participant-Komponente | Am Actor | Identität im Level, persistentes Gedächtnis |
| Subsystem | In der Welt (automatisch) | Gespräche starten, verwalten, beenden |
| UI-Schicht | Während eines Dialogs | Text, Portraits, Choices anzeigen |

Weiter: [Graph & visuelle Sprache](graph-visual-language.md).
