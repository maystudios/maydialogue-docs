---
description: Breakpoints setzen, Step Over/Into/Out nutzen und Variablen live im Watch-Panel beobachten.
---

# Debugger & Breakpoints

Der PIE-Debugger ist dein Werkzeug, wenn ein Dialog im echten Spiel nicht das tut, was er soll. Er verhält sich wie der Blueprint-Debugger – nur für Dialoge.

Nutze den [Preview-Runner](preview-runner.md) für schnelle Text-Iteration. Den Debugger nutze, wenn du den echten GAS-State, Quest-Zustand oder NPC-Verhalten in der Schleife brauchst.

## Setup in drei Schritten

1. Dialog im Asset-Editor öffnen.
2. Breakpoints auf relevanten Nodes setzen (Rechtsklick → **Toggle Breakpoint** oder `F9`).
3. **PIE starten** und den Dialog im Spiel auslösen.

Sobald die Ausführung einen Node mit Breakpoint erreicht:
- Der Dialog pausiert.
- Das Editor-Fenster kommt in den Vordergrund.
- Der Graph zentriert auf den pausierten Node.
- Der Node leuchtet in der **Aktiv-Debug-Farbe** (Standard: Gelb).

> 📸 **Bild-Platzhalter:** `debugger-breakpoint-hit.png` — Graph mit pausiertem Branch-Node, gelb hervorgehoben. Watch-Panel unten sichtbar.
> *Setup:* PIE läuft, Dialog ausgelöst. Branch-Node „CheckDialogueVariable(AngerLevel >= 3)" mit Breakpoint. Node leuchtet gelb (ActiveDebugColor). Roter Pfeil auf den leuchtenden Node. Im Hintergrund DebuggerWatch-Tab unten mit `AngerLevel = 2` sichtbar.

## Breakpoints verwalten

### Setzen und entfernen

| Methode | Wirkung |
| --- | --- |
| `F9` (Node selektiert) | Breakpoint togglen |
| Rechtsklick auf Node → Toggle Breakpoint | Breakpoint togglen |

Ein gesetzter Breakpoint erscheint als **roter Punkt oben rechts am Node**.

### Deaktivieren (ohne löschen)

Rechtsklick auf den Breakpoint-Indikator → **Disable**. Der Breakpoint bleibt erhalten, wird aber nicht ausgelöst. Nützlich, wenn du einen Bereich vorübergehend ohne Unterbrechung durchlaufen willst.

### Alle Breakpoints deaktivieren

**Debug-Menü → Disable All Breakpoints**. Alle Breakpoints im Asset werden deaktiviert, aber nicht gelöscht. Mit **Enable All Breakpoints** reaktivieren.

{% hint style="info" %}
Breakpoints werden **pro Benutzer** im Projekt gespeichert und überleben Editor-Neustarts. Du musst sie nicht nach jedem Neustart neu setzen.
{% endhint %}

## Step-Controls

Nach einem Breakpoint-Hit stehen diese Aktionen zur Verfügung:

| Action | Shortcut | Wirkung |
| --- | --- | --- |
| **Continue** | `F5` | Weiter bis zum nächsten Breakpoint |
| **Step Over** | `F10` | Aktuellen Node ausführen, am nächsten halten |
| **Step Into** | `F11` | Bei Link-/SubGraph-Nodes: in den verlinkten Graph springen |
| **Step Out** | `Shift+F11` | Aus aktuellem Sub-Graph/Link-Scope herausspringen |

> 📸 **Bild-Platzhalter:** `debugger-step-controls.png` — Toolbar oder Debug-Menü mit Step-Buttons annotiert.
> *Setup:* PIE-Debugger aktiv (Breakpoint getroffen). Debug-Toolbar zeigt Continue (F5), Step Over (F10), Step Into (F11), Step Out (Shift+F11). Rote Pfeile mit Beschriftung auf jeden Button.

## Aktiv-Highlighting im Graph

Während einer Debug-Session:

| Farbe | Bedeutung |
| --- | --- |
| **Gelb** (ActiveDebugColor) | Aktuell pausierter Node |
| **Blassblaues** (HistoryDebugColor) | Bereits besuchte Nodes in dieser Session |

So siehst du auf einen Blick, welchen Pfad der Dialog genommen hat.

> 📸 **Bild-Platzhalter:** `debugger-history-highlight.png` — Graph mit pausiertem Node (gelb) und drei vorher besuchten Nodes (blassblau).
> *Setup:* Debug-Session: Entry (blassblau), SayLine Guard (blassblau), PlayerChoice (blassblau), Branch (gelb, aktuell pausiert). Verlauf des Dialogs gut erkennbar. Legende-Annotation im Bild: gelb = aktuell, blassblau = besucht.

## Watch-Panel

Der **DebuggerWatch**-Tab (unten) zeigt während der Pause:

- **Dialogue-Variablen** der aktiven Instance mit aktuellem Wert
- **Participant-Variablen** aller teilnehmenden Actors
- Datentyp und String-Darstellung des Werts

Du kannst Werte **direkt im Watch-Panel editieren**. Ändere einen Wert, drücke `F5` (Continue) – die Ausführung läuft mit dem neuen Wert weiter.

> 📸 **Bild-Platzhalter:** `debugger-watch-panel.png` — DebuggerWatch-Tab mit vier Variablen, eine wird gerade editiert.
> *Setup:* DebuggerWatch-Tab unten aktiv. Vier Zeilen: `HasAskedName` (Bool, false), `AngerLevel` (Int, 2 – gerade im Edit-Modus, Eingabefeld offen mit Cursor), `HasMet` (Bool, true, Participant-Scope), `Friendship` (Float, 0.7, Participant-Scope). Roter Pfeil auf das editierbare `AngerLevel`-Feld.

## Typische Debug-Workflows

### „Warum wird Choice 2 nicht angezeigt?"

1. Breakpoint auf den PlayerChoice-Node.
2. PIE starten, Dialog auslösen.
3. Beim Hit: Watch-Panel prüfen – welcher Tag oder Variable fehlt?
4. Wert im Watch-Panel auf den erwarteten setzen, `F5` drücken.
5. Prüfen ob die Choice jetzt erscheint.

### „Welchen Output nimmt der Branch?"

1. Breakpoint auf den Branch-Node.
2. Beim Hit: `F10` (Step Over).
3. Der nächste Highlight-Node zeigt den gewählten Ausgang.

### „Was passiert nach dem FireEvent?"

1. Breakpoint auf den FireEvent-Node.
2. Beim Hit: Watch-Panel notieren, `F10` (Step Over).
3. Beobachten, ob der folgende Wait-Node aufwacht.

### „Läuft der SubGraph korrekt?"

1. Breakpoint auf den SubGraph-Node.
2. Beim Hit: `F11` (Step Into) – du springst in den Sub-Graphen.
3. Dort weiter steppen.
4. `Shift+F11` (Step Out) um in den Eltern-Graphen zurückzukehren.

## Debugger und Preview kombinieren

Die beiden Tools ergänzen sich optimal:

| Phase | Tool |
| --- | --- |
| Text, Choices, Variablen schreiben und testen | Preview-Runner |
| GAS-Tags, Quest-State, echte NPC-Logik validieren | PIE-Debugger |

Typischer Ablauf: 90% der Arbeit im Preview-Runner, abschließende Validierung im Debugger.

## Bekannte Grenzen

- **Cross-Asset Step-Into:** Wenn ein Link-Node in ein anderes Asset springt, öffnet der Editor das Ziel-Asset nicht automatisch. Workaround: Ziel-Asset manuell öffnen, dort Breakpoint setzen. Den Link-Target findest du im Details-Panel des Link-Nodes im Quell-Asset (Property `TargetAsset`).
- **Async-Nodes:** Wait- und PlayAnimation-Nodes pausieren erst, wenn sie beendet sind – nicht mittendrin.

Weiter: [Komfort-Features →](comfort-features.md)
