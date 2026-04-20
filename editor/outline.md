# Outline

Der **Outline-Tab** ist die Liste-Sicht auf deinen Dialog. Während der Graph die räumliche Anordnung zeigt, zeigt der Outline die **lineare Struktur** – Sprecher, Text-Previews, Typ-Badges.

## Warum eine Liste, wenn es einen Graph gibt?

Der Graph ist räumlich – du suchst Nodes nach ihrer Position. Dialoge denken aber in **Sprechern und Zeilen**, nicht in 2D-Koordinaten. *„Wo steht die SayLine, in der Anna vom Keller spricht?"* ist eine Sprecher+Text-Suche, keine Positions-Suche.

Der Outline ist die direkte Antwort darauf:

```
🔴 Wächter    │  Halt! Wer bist du?                          │ Say
🔵 Spieler    │  Du antwortest:                              │ PC
🔴 Wächter    │  Dann passiere in Frieden.                   │ Say
🔴 Wächter    │  Dann verzieh dich!                          │ Say
              │  (Exit: Completed)                           │ X
              │  (Exit: Failed)                              │ X
```

## Anatomie

Pro Zeile:

| Element | Bedeutung |
| --- | --- |
| **Farb-Chip** | `NodeColor` des Sprechers (grau bei nicht-Sprecher-Nodes). |
| **Primärtext** | DisplayName (Sprecher) oder Node-Titel. |
| **Sekundärtext** | Gekürzter Dialog-Text (max 60 Zeichen, ellipsis). |
| **Typ-Badge** | Zwei-Buchstaben-Kennung rechts. |

### Typ-Badges

| Badge | Bedeutung |
| --- | --- |
| `Say` | SayLine |
| `PC` | PlayerChoice |
| `Br` | Branch |
| `Rn` | RandomLine |
| `Wt` | Wait |
| `Ln` | Link |
| `Sg` | SubGraph |
| `Ev` | FireEvent |
| `Fx` | ApplyEffect |
| `SV` | SetVariable |
| `PS` | PlaySound |
| `CF` | CameraFocus |
| `CS` | CameraShake |
| `PA` | PlayAnimation |
| `E` | Entry |
| `X` | Exit |

## Bedienung

### Suche

Suchfeld am oberen Rand. Filtert **live** nach:

* Primärtext (Sprecher-Name / Node-Titel).
* Sekundärtext (Dialog-Text, gekürzt).
* Typ-Badge (z.B. *„Say"* zeigt alle SayLines).

### Filter

Dropdown rechts neben dem Suchfeld. Optionen:

* **All** (Default).
* **Say Line**.
* **Player Choice**.
* **Branch**.
* **Actions** (alle Action-Nodes).
* **Flow Control** (Wait, Link, SubGraph).
* **Special** (Entry, Exit, Knot).

Kombinierbar mit der Suche.

### Click-to-Jump

Klick auf eine Zeile selektiert den Node im Graph und zentriert die Kamera auf ihn. Perfekt, um in einem 150-Node-Asset einen bestimmten Moment zu finden.

### Live-Update

Das Panel bindet sich an das Graph-Change-Delegate. Nodes hinzufügen / löschen / ändern aktualisiert die Liste sofort.

{% hint style="info" %}
Falls das Delegate in seltenen Fällen nicht feuert, hat der Outline ein **Polling-Fallback** (Hash-Check pro Tick) – die Liste wird so oder so synchron gehalten.
{% endhint %}

## Einsatzfälle

### „Wer sagt das?"

Such-Feld: Textauszug eingeben. Die erste Treffer-Zeile zeigt dir den Sprecher-Chip und den Namen.

### „Wie viele SayLines hat dieser Dialog?"

Filter auf **Say Line**, Zeilenzahl ablesen.

### „Welche Choices hat der Dialog?"

Filter auf **Player Choice**, alle Choice-Nodes werden gelistet.

### „Geh zum dritten RandomLine-Node"

Filter auf **Action** oder direkt über Typ-Badge-Suche (`Rn`), dritte Zeile anklicken.

## Minimap-Ersatz

Die Design-Idee: Dialog-Autoren denken in Sprechern und Zeilen, nicht in 2D-Positionen. Eine pixelige Minimap hilft weniger als eine gut durchsuchbare Liste.

Deshalb gibt es aktuell **keine Minimap** als eigenen Tab. Falls du trotzdem eine räumliche Übersicht brauchst, nutze den Graph-Zoom-Out und die Auto-Layout-Aktion.

Weiter: [Find-in-Dialogue →](find.md).
