---
description: "Der schnellstmögliche erste Dialog — und die Editor-Komfort-Features, die dich dorthin bringen."
---

# Onboarding & Editor-Komfort

Im [Quick Start](quick-start.md) hast du gerade einen Dialog auf dem langen Weg gebaut. Diese Seite zeigt den **schnellen** Weg: das Scaffold, die Templates und die Editor-Hinweise, die dich von „neues Projekt" zu „sprechendem NPC in PIE" in etwa fünf Minuten bringen — das meiste davon, ohne irgendetwas zu schreiben oder zu verdrahten.

Wenn du nur eine Onboarding-Seite liest, dann diese. Sie räumt außerdem die häufigste Erstnutzer-Verwirrung aus dem Weg (die UMG-Widget-Falle), bevor sie überhaupt zuschnappt.

{% hint style="success" %}
**Wo das hingehört:** Mach den [Quick Start](quick-start.md) einmal, um die beweglichen Teile zu verstehen, und nutze danach jedes Mal die Abkürzungen hier. Wenn du die volle Feature-Tour willst — Variablen, GAS, Requirements — geh zum [Walkthrough](first-dialogue.md).
{% endhint %}

---

## Deine ersten 5 Minuten

Ein geführter Weg, der das manuelle Graph-Verdrahten überspringt:

1. **Dialog-Asset anlegen** — es öffnet sich vorverdrahtet mit einem Starter-Scaffold, nicht als leerer Graph.
2. **Einen „Dialogue NPC" ins Level ziehen** aus dem Place-Actors-Menü.
3. **Deinen Dialog zuweisen** im `DefaultDialogue`-Slot des NPC.
4. **Play drücken.** Hingehen, reden.

Jeder Schritt wird unten ausführlich beschrieben.

> 📸 **Bild-Platzhalter:** `onboarding-five-minute-flow.png` — Der vierstufige Onboarding-Ablauf als Strip.
> *Setup:* Einfache horizontale Grafik mit vier beschrifteten Boxen von links nach rechts: „Neues Asset (scaffolded)" → „Dialogue NPC ziehen" → „DefaultDialogue zuweisen" → „Play drücken". Pfeile dazwischen. Bildunterschrift: „Von neuem Projekt zum sprechenden NPC in vier Schritten."

---

## Schritt 1: Neue Assets öffnen mit Starter-Scaffold

Wenn du ein neues **May Dialogue → Dialogue Asset** anlegst, öffnet es sich nicht mehr als leere Leinwand mit einem einsamen Entry-Node. Es öffnet sich mit einem funktionsfähigen **Entry → SayLine → Exit**-Scaffold, das bereits platziert und verbunden ist.

Das heißt, ein brandneues Asset ist **bereits gültig und sofort lauffähig**: es compiliert, es hat eine Startzeile, die du an Ort und Stelle bearbeiten kannst, und es endet sauber. Du ersetzt den Platzhalter-Text auf der SayLine, drückst Compile, und du hast einen Einzeiler-Dialog — kein Node-für-Node-Verdrahten nötig.

> 📸 **Bild-Platzhalter:** `onboarding-starter-scaffold.png` — Ein frisch angelegtes Asset mit dem vorverdrahteten Scaffold.
> *Setup:* Asset-Editor offen auf einem gerade erstellten `DA_New_Dialogue`. Der Graph zeigt Entry (grün) → SayLine (mit dem gesetzten Platzhalter-Text „Hello! Edit me in the Details panel.") → Exit (rot), alle mit sichtbaren Drähten verbunden. Bildunterschrift: „Neue Assets sind von der ersten Sekunde an lauffähig."

{% hint style="info" %}
Das Scaffold ist ein ganz normaler Graph. Lösche, verdrahte um oder erweitere es wie jeden anderen — sobald das Asset existiert, ist an den Starter-Nodes nichts Besonderes.
{% endhint %}

---

## Schritt 2: Das „Dialogue NPC"-Template in Place Actors

Öffne das **Place-Actors**-Panel (Window → Place Actors) und suche nach **Dialogue NPC** — oder öffne den dedizierten **MayDialogue**-Kategorie-Tab, in dem das Template registriert ist. Zieh es ins Level.

Das ist ein fertiger Actor, der bereits eine `MayDialogueParticipant`-Komponente trägt — du musst die Komponente also nicht hinzufügen, einrichten oder wissen, welche Felder wichtig sind. Er landet als sprechbarer NPC out of the box.

> 📸 **Bild-Platzhalter:** `onboarding-place-actors-dialogue-npc.png` — Das Place-Actors-Panel mit dem Dialogue-NPC-Template.
> *Setup:* Editor mit geöffnetem Place-Actors-Panel, „Dialogue" ins Suchfeld getippt, der **Dialogue NPC**-Eintrag in der Ergebnisliste hervorgehoben. Ein roter Pfeil zeigt auf den Eintrag. Bildunterschrift: „Zieh einen sprechbaren NPC direkt ins Level."

---

## Schritt 3: `DefaultDialogue` zuweisen

Wähle den gerade platzierten Dialogue NPC aus. Im **Details**-Panel findest du seine `MayDialogueParticipant`-Komponente und setzt **`DefaultDialogue`** auf das Asset aus Schritt 1.

Das ist das einzige Feld, das du setzen musst. Der Participant-Tag und der Display-Name kommen auf dem Template vorausgefüllt; du kannst sie später überschreiben, wenn du mehr als einen NPC hast.

> 📸 **Bild-Platzhalter:** `onboarding-assign-default-dialogue.png` — Die Participant-Komponente mit zugewiesenem DefaultDialogue.
> *Setup:* Details-Panel des platzierten Dialogue NPC, die MayDialogueParticipant-Komponente aufgeklappt, `DefaultDialogue` auf `DA_New_Dialogue` gesetzt. Ein roter Pfeil zeigt auf das DefaultDialogue-Feld. Bildunterschrift: „Ein Feld verbindet den NPC mit seinem Dialog."

---

## Schritt 4: Play drücken

Starte PIE, geh zum NPC und löse ihn aus. Das Slate-Debug-Widget erscheint, der NPC spricht deine Zeile, und du hast einen funktionierenden Dialog ohne ein einziges Blueprint und ohne Code.

Wenn du den Dialog aus deiner eigenen Logik auslösen willst (ein Interaktions-Prompt, ein Trigger-Volume) statt dich auf den Default des Templates zu verlassen, folge den Trigger-Optionen in [Quick Start → Schritt 9](quick-start.md#schritt-9-dialog-auslosen).

---

## Der Quick-Start-Assistent

MayDialogue liefert einen leichtgewichtigen **Quick-Start-Assistenten**, kein vollständiges interaktives Schritt-für-Schritt-Tutorial. Es sind zwei kleine Hilfen, die dich auf diese Dokumentation hinweisen:

- **Eine Orientierungs-Benachrichtigung beim ersten Öffnen.** Der erste Dialog-Editor, den du in einer Session öffnest — und nur, wenn das Asset noch wie das frisch erstellte Starter-Scaffold aussieht — zeigt einen einmaligen Toast: *„New here? The graph already contains a starter scaffold — double-click the Say Line to edit its text."* Er trägt einen **Open-Quick-Start**-Link, der diese Dokumentation in deinem Browser öffnet. Er ist fire-and-forget (er verschwindet von selbst) und erscheint höchstens einmal pro Editor-Session.
- **Ein Help-Menü-Eintrag.** Das **Help**-Menü des Asset-Editors hat einen **Open Quick Start Guide**-Eintrag, der diese Dokumentation jederzeit öffnet.

Das ist der gesamte Assistent — ein Orientierungs-Anstoß plus ein Deep-Link, kein Overlay, das dich durch die Panels führt. Der praktische Schritt-für-Schritt-Durchlauf ist der geschriebene [Quick Start](quick-start.md), auf den der Assistent dich verweist.

{% hint style="info" %}
Es gibt in 1.0 bewusst **kein** interaktives In-Editor-Tutorial-Overlay — ein vollständiger geführter Durchlauf wurde als zu schwergewichtig descoped. Das Starter-Scaffold plus diese Dokumentation sind der Onboarding-Pfad.
{% endhint %}

> 📸 **Bild-Platzhalter:** `onboarding-quickstart-notification.png` — Die Orientierungs-Benachrichtigung beim ersten Öffnen.
> *Setup:* Asset-Editor offen auf einem frisch erstellten Asset. Ein Slate-Notification-Toast in der Ecke liest „New here? The graph already contains a starter scaffold — double-click the Say Line to edit its text." mit einem „Open Quick Start"-Link. Bildunterschrift: „Ein einmaliger Anstoß verweist dich auf die Quick-Start-Docs."

---

## Die Dokumentation aus dem Editor öffnen

Der Dialog-Asset-Editor verlinkt diese Dokumentation an zwei Stellen direkt, sodass die Referenz nie mehr als einen Klick entfernt ist, wenn du mitten in einer Aufgabe steckst:

- Das **Help**-Menü → **Open Quick Start Guide**.
- Der eingebaute **Documentation**-Link des Editors (die Standard-Hilfe-Affordance des Asset-Editors), verdrahtet auf dieselbe URL.

Beide öffnen `https://maystudios.net/maydialogue/docs` in deinem Standard-Browser.

> 📸 **Bild-Platzhalter:** `onboarding-help-menu-quickstart.png` — Das Help-Menü des Asset-Editors mit dem Quick-Start-Eintrag.
> *Setup:* Oberer Bereich des Dialog-Asset-Editors mit geöffnetem **Help**-Menü, der **Open Quick Start Guide**-Eintrag hervorgehoben. Ein roter Pfeil zeigt darauf. Bildunterschrift: „Die Dokumentation ist einen Klick vom Editor entfernt."

---

## Die UMG-Widget-Falle — behoben

Das hat früher fast jeden am ersten Tag verwirrt, deshalb lohnt es sich zu verstehen.

**Die alte Verwirrung:** MayDialogue liefert zwei UI-Schichten — das immer aktive **Slate-Debug-Widget** und ein optionales **UMG-Widget**, das du über `DefaultDialogueWidgetClass` zuweist. Früher hat das Setzen deines eigenen UMG-Widgets das Slate-Debug-Widget **nicht** abgeschaltet. Das Ergebnis: Leute wiesen ein schönes eigenes Widget zu, drückten Play und sahen **beide** — ihr Widget *und* das schlichte Debug-Widget übereinander gestapelt — ohne erkennbaren Grund.

**Der Fix:** Das Setzen von `DefaultDialogueWidgetClass` **deaktiviert jetzt automatisch das Slate-Debug-Widget** (und die Settings-UI zeigt einen klaren Hinweis, der den Zusammenhang erklärt). Weise dein UMG-Widget zu und das Debug-Widget tritt automatisch zur Seite — kein zweiter Schalter, den man sich merken muss.

| Du setzt | Was angezeigt wird | Hinweis |
|---|---|---|
| Nichts (Standard) | Slate-Debug-Widget | Funktioniert out of the box zum Testen. |
| `DefaultDialogueWidgetClass` = dein UMG-Widget | Nur dein UMG-Widget | Slate-Debug-Widget automatisch deaktiviert; ein Hinweis bestätigt das. |

{% hint style="warning" %}
Wenn du auf einem älteren Build bist und zwei Widgets gleichzeitig siehst, ist das die ursprüngliche Falle. Update, oder deaktiviere `bUseSlateDialogueWidget` manuell in **Project Settings → Plugins → MayDialogue → Widget**.
{% endhint %}

> 📸 **Bild-Platzhalter:** `onboarding-umg-hint.png` — Der Settings-Hinweis, der erscheint, wenn ein UMG-Widget zugewiesen ist.
> *Setup:* Project Settings → Plugins → MayDialogue → Widget. `DefaultDialogueWidgetClass` ist auf ein eigenes Widget gesetzt; darunter liest ein Info-Hinweis etwas wie „Ein UMG-Widget ist zugewiesen — das Slate-Debug-Widget ist automatisch deaktiviert." Ein roter Pfeil zeigt auf den Hinweis. Bildunterschrift: „Weise ein UMG-Widget zu und das Debug-Widget tritt von selbst zur Seite."

---

## Edge-Cases & Stolperfallen

- **Das Scaffold ist eine Bequemlichkeit, keine Vorschrift.** Wenn dein Studio-Template leere Assets erwartet, lösche die Scaffold-Nodes nach dem Anlegen — nichts hängt von ihnen ab.
- **Das Dialogue-NPC-Template ist ein Startpunkt.** Es ist ein einfacher Actor mit einer Participant-Komponente; reparente ihn zu deiner eigenen Character-Klasse oder kopiere die Komponente auf einen bestehenden Pawn, wenn du darüber hinauswächst.
- **`DefaultDialogue` ist nur für den Default-Trigger-Pfad des Templates relevant.** Wenn du Dialoge explizit über den Participant oder die Library startest, kannst du es leer lassen und das Asset zur Aufrufzeit übergeben — siehe [Dialog starten](../runtime/starting-dialogues.md).
- **Die UMG-Auto-Deaktivierung greift bei `DefaultDialogueWidgetClass`, nicht bei Pro-Aufruf-Widgets.** Wenn du ein Dialog-Widget selbst über einen Nicht-Standard-Pfad spawnst, verwalte die Sichtbarkeit des Slate-Debug-Widgets selbst.
- **Die Docs-Links brauchen eine Netzwerkverbindung**, um die Online-Dokumentation zu erreichen.

---

## Was als Nächstes?

* Einen echten, verzweigenden Dialog mit Variablen und Bedingungen bauen → [Walkthrough](first-dialogue.md)
* Defaults einstellen (Typewriter-Tempo, Advance-Modus, Audio) → [Projekt-Einstellungen](project-settings.md)
* Das Slate-Debug-Widget gegen eine eigene UMG-Schicht tauschen → [UI-Architektur](../ui/umg-architecture.md)
* Den Dialog barrierefrei machen (größerer Text, Screen-Reader) → [Barrierefreiheit](../appendix/accessibility.md)
