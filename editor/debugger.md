# Debugger & Breakpoints

Für alles, was der [Preview-Runner](preview-runner.md) nicht abbildet – echte NPC-Physik, GAS-State, Welt-Logik – nutzt du den **PIE-Debugger**. Er verhält sich wie der Blueprint-Debugger, nur für Dialoge.

## Start

1. Dialog im Asset-Editor offen.
2. Beliebige Nodes per Rechtsklick → **Toggle Breakpoint** mit Breakpoints versehen (`F9`).
3. **PIE starten** (gewöhnliches Play-in-Editor).
4. Dialog in PIE auslösen.

Sobald die Ausführung einen Node mit Breakpoint erreicht:

* Dialog pausiert.
* Editor-Fenster kommt in den Vordergrund.
* Graph zentriert auf den pausierten Node.
* Node-Body pulsiert in `ActiveDebugColor` (siehe Editor-Settings).

## Step-Controls

Nach einem Breakpoint-Hit:

| Action | Wirkung |
| --- | --- |
| **Continue** (`F5`) | Weiter bis zum nächsten Breakpoint. |
| **Step Over** (`F10`) | Aktuellen Node ausführen, am nächsten Node halten. |
| **Step Into** (`F11`) | Bei Link-/SubGraph-Nodes: in den verlinkten Graph springen. |
| **Step Out** (`Shift+F11`) | Aus dem aktuellen Scope (Sub-Graph / Link) herausspringen. |

## Breakpoint-Verwaltung

### Toggle

* `F9` oder Rechtsklick → **Toggle Breakpoint** auf einem Node.

### Enable/Disable (ohne Löschen)

Ein Breakpoint kann **deaktiviert** werden – er bleibt im Asset erhalten, wird aber nicht ausgelöst:

* Rechtsklick auf Breakpoint-Indikator → **Enable** / **Disable**.

### All Breakpoints Off

Toolbar-Menü **Debug → Disable All Breakpoints**. Nützlich, wenn du kurz ohne Unterbrechungen testen willst.

### Persistence

Breakpoints werden **pro User** im Projekt gespeichert (EditorPerProjectUserSettings) und überleben Editor-Neustarts.

## Aktiv-Highlighting

Während der Debug-Session:

* Der **aktuell pausierte Node** leuchtet in `ActiveDebugColor` (Default: Gelb).
* **Bereits besuchte Nodes** in `HistoryDebugColor` (Default: Blasses Blau).

So siehst du auf einen Blick, welchen Weg der Dialog genommen hat.

## Watch-Panel

Der **DebuggerWatch**-Tab zeigt während der Pause:

* **Dialogue-Variablen** der aktiven Instance.
* **Participant-Variablen** aller teilnehmenden Participants.
* Datentyp und aktueller String-Wert.

Felder sind **editierbar** – du kannst Werte live setzen und beim nächsten Continue die Auswirkungen beobachten.

## Typische Workflow-Muster

### „Warum wird Choice 2 nicht angezeigt?"

1. Breakpoint auf den PlayerChoice-Node.
2. PIE starten, Dialog auslösen.
3. Hit → Watch-Panel zeigt Variablen.
4. Prüfen: Welcher Tag / Attribut / Variable-Zustand liegt vor?
5. Gegebenenfalls Wert in Watch editieren, Continue klicken.

### „Was passiert nach dem FireEvent?"

1. Breakpoint auf den FireEvent-Node.
2. PIE starten, Dialog triggern.
3. Bei Hit → Step Over, beobachte, ob ein Wait-Node aufwacht.

### „Welchen Output nimmt der Branch?"

1. Breakpoint auf den Branch-Node.
2. Bei Hit → Step Over. Das nächste Highlight zeigt den gewählten Ausgang.

## Bekannte Einschränkungen

* **Cross-Asset-Step-Into ist rudimentär.** Wenn ein Link-Node in ein anderes Asset springt, öffnet der Editor das neue Asset derzeit nicht automatisch. Workaround: Asset manuell öffnen, Breakpoint dort setzen.
* **Async-Nodes pausieren nicht sauber.** Wait-/PlayAnimation-Nodes sind als „running" markiert, aber der Breakpoint feuert erst bei deren Beendigung – so kannst du nicht mitten in der Animation pausieren.

## Debugger + Preview kombinieren

Die beiden Tools ergänzen sich:

* **Preview**: schnelle Text- und Choice-Iteration.
* **Debugger**: validierung der echten Game-Kombination.

Typisches Muster: 90% im Preview schreiben, abschließend im Debugger mit echten GAS-Tags / Quest-State verifizieren.

Weiter: [Komfort-Features →](comfort-features.md).
