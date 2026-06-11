---
description: Größerer Text, farbenblind-sichere Choice-Marker, Screen-Reader-Unterstützung und remappbarer Input — alles eingebaut.
---

# Barrierefreiheit

MayDialogue liefert seine Dialog-UI mit eingebauter Barrierefreiheit aus, damit die Spieler, die sie brauchen, nicht von deiner Geschichte ausgeschlossen werden. Diese Seite bündelt alle Barrierefreiheits-Features an einem Ort: wie du sie einschaltest, wo jede Einstellung liegt und was sie unter der Haube genau tut.

Vier Bereiche werden abgedeckt:

1. **Größerer Text** — ein projektweiter `DialogueTextScale` von 0,5× bis 3,0×, angewendet auf **beiden** UI-Schichten.
2. **Farbenblind-sichere Choice-Marker** — ein farbunabhängiger Schloss-Marker auf nicht verfügbaren Choices, standardmäßig an.
3. **Screen-Reader-Unterstützung** — Accessible-Text für Sprecher-Name, Dialogtext und jeden Choice-Button, auf beiden Schichten.
4. **Input-Remapping** — über Enhanced Input gesteuert, sodass Spieler die Tasten für Weiter/Skip/Auswahl neu belegen können.

Alles hier funktioniert auf dem **eingebauten Slate-Debug-Widget** und auf den **Standard-UMG-Komponenten-Widgets** ohne eine Zeile Code. Wenn du ein komplett eigenes UMG-Widget von Grund auf baust, sind dieselben Helfer öffentlich, sodass du sie selbst einbinden kannst — siehe API-Referenz unten.

{% hint style="info" %}
Das sind die Features hinter SPEC §5.11. Sie sind so ausgelegt, dass sie dem Anspruch eines kommerziellen Plugins genügen, dessen Zielgruppe von Solo-Indies bis zu professionellen Studios reicht — Barrierefreiheit ist ein Auslieferungs-Feature, kein nachträglicher Gedanke.
{% endhint %}

---

## Der Weg ohne Code

Öffne **Edit → Project Settings → Plugins → MayDialogue → Accessibility**. Zwei Schalter decken die visuellen Features ab; der Rest läuft automatisch.

1. Setze **Dialogue Text Scale** auf die gewünschte Größe (z.B. `1.5` für 150% Text). Sie greift bei der nächsten angezeigten Dialogzeile.
2. Lass **Colorblind Safe Choice Cues** aktiviert (Standard). Gesperrte Choices zeigen jetzt einen 🔒-Schloss-Marker vor dem Text.
3. Screen-Reader-Text wird von beiden Widget-Schichten automatisch ausgegeben — es gibt nichts zu aktivieren.
4. Input-Remapping kommt aus deinem Enhanced-Input-Mapping-Context (siehe [Input-Remapping](#input-remapping-enhanced-input)).

Das ist das gesamte Setup. Kein Blueprint, kein C++, kein Widget-Austausch.

> 📸 **Bild-Platzhalter:** `accessibility-settings-panel.png` — Die Accessibility-Kategorie der MayDialogue-Projekt-Einstellungen.
> *Setup:* Editor offen, Edit → Project Settings → Plugins → MayDialogue, gescrollt zur **Accessibility**-Kategorie. Beide Zeilen sichtbar: `Dialogue Text Scale` (Slider/Feld auf 1.5) und `Colorblind Safe Choice Cues` (aktiviert). Ein roter Pfeil zeigt auf die Kategorie-Überschrift.

---

## Größerer Text — `DialogueTextScale`

`DialogueTextScale` ist ein einziger einheitlicher Multiplikator, der auf **jede** Dialog-Schriftgröße angewendet wird: den Nachrichtentext, den Sprecher-Namen und die Choice-Button-Beschriftungen. Er ist auf den Bereich **0,5–3,0** begrenzt (Werte außerhalb werden still in den Bereich gebracht).

Der Wert wird über den gemeinsamen Helfer `MayDialogue::Accessibility::ClampTextScale(...)` ausgelesen und begrenzt, sodass das **Slate-Debug-Widget** und die **Standard-UMG-Komponenten-Widgets** exakt dieselben Grenzen anwenden — kein Widget driftet auf einen anderen Scale. Du musst kein Widget ersetzen, um größeren Text zu verwenden: setze den Scale einmal und er gilt projektweit.

{% hint style="info" %}
`DialogueTextScale` wird auf **beiden** Schichten berücksichtigt: Das Slate-Debug-Widget und die Standard-UMG-Komponenten-Widgets (Text, Speaker, ChoiceButton) lesen ihn alle über `ClampTextScale`.
{% endhint %}

```ini
# DefaultGame.ini — 150% Dialogtext überall
[/Script/MayDialogue.MayDialogueSettings]
DialogueTextScale=1.5
```

**Eigene Widgets:** Wenn du ein eigenes UMG-Widget geschrieben hast, das nicht von den Standard-Komponenten-Widgets erbt, lies `DialogueTextScale` aus den Settings und schicke ihn durch `ClampTextScale`, bevor du ihn auf deine Text-Blöcke anwendest — so bleibst du im unterstützten Bereich und konsistent mit den eingebauten Widgets.

> 📸 **Bild-Platzhalter:** `accessibility-text-scale-compare.png` — Dieselbe Dialogzeile bei 1,0× und 2,0× nebeneinander.
> *Setup:* Zwei PIE-Screenshots nebeneinander derselben SayLine ("Halt! Wer bist du?") mit zwei Choices. Links: `DialogueTextScale=1.0`. Rechts: `DialogueTextScale=2.0`. Das rechte Panel zeigt sichtbar größeren Nachrichtentext, Sprecher-Namen und Choice-Buttons. Bildunterschrift: „Eine Einstellung skaliert die gesamte Dialog-UI."

---

## Farbenblind-sichere Choice-Marker — `bColorblindSafeChoiceCues`

Wenn eine Choice nicht verfügbar ist (ihr Requirement gab `FailedButVisible` zurück), dimmt die UI sie normalerweise ab. Dimmen allein verlässt sich auf Farb-/Kontrastwahrnehmung. Mit aktiviertem `bColorblindSafeChoiceCues` (**Standard**) erhält jede nicht-verfügbar-aber-sichtbare Choice einen **Schloss-Glyph-Präfix** (🔒, `U+1F512`) vor ihrem Text, sodass der gesperrte Zustand ganz ohne Farbwahrnehmung lesbar ist.

Der Präfix wird von `MayDialogue::Accessibility::GetColorblindChoicePrefix(bEnabled, bIsAvailable)` erzeugt und **vor** den Choice-Text gesetzt, sodass er in jedem Theme unabhängig von der Button-Farbe sichtbar ist. Beide Widget-Schichten nutzen denselben Helfer, sodass verfügbare/gesperrte Choices über Slate und UMG identisch aussehen.

Deaktiviere ihn nur, wenn dein Projekt bereits einen eigenen farbunabhängigen Verfügbarkeits-Indikator hat (ein separates Schloss-Icon-Widget, ein Durchstreichen usw.):

```ini
# DefaultGame.ini — farbenblind-sicheren Schloss-Marker an lassen (Standard)
[/Script/MayDialogue.MayDialogueSettings]
bColorblindSafeChoiceCues=True
```

> 📸 **Bild-Platzhalter:** `accessibility-choice-lock-glyph.png` — Eine Choice-Liste mit einer verfügbaren und einer gesperrten Choice.
> *Setup:* PIE-Screenshot eines PlayerChoice mit zwei Einträgen. Eine Choice verfügbar („Den Zoll zahlen"), eine gesperrt, weil ein Requirement fehlschlägt („Den Wächter überreden"). Die gesperrte Choice zeigt den 🔒-Präfix vor ihrem Text. Bildunterschrift: „Gesperrte Choices bleiben unterscheidbar, ohne sich auf Farbe zu verlassen."

---

## Screen-Reader-Unterstützung

Beide Widget-Schichten geben **Accessible-Text** für die drei Dinge aus, die ein Screen-Reader-Nutzer braucht, um einem Dialog zu folgen: wer spricht, was gesagt wurde und welche Choices es gibt. Der Text wird von drei gemeinsamen Helfern gebaut, sodass die Formulierung auf Slate und UMG identisch ist:

| Element | Helfer | Angesagt als |
|---|---|---|
| Sprecher-Name | `MakeSpeakerAccessibleText(SpeakerName)` | `Speaker: <Name>` |
| Dialogtext | `MakeDialogueTextAccessibleText(MessageText)` | der reine (rich-tag-bereinigte) Satz |
| Verfügbare Choice | `MakeChoiceAccessibleText(...)` | `Choice 1: <Text>` |
| Gesperrte Choice | `MakeChoiceAccessibleText(...)` | `Locked choice 1: <Text>` (plus `— <Grund>`, wenn ein Grund gesetzt ist) |

Das entscheidende Detail für assistive Technologie: die **Locked**-Ansage enthält die Worte „Locked choice" und, wenn das Requirement einen liefert, die Anforderungsbeschreibung — so hört ein Screen-Reader-Nutzer, *warum* eine Choice nicht verfügbar ist, obwohl er das 🔒-Glyph oder die abgedimmte Farbe nicht sehen kann. Der Dialogtext ist immer der **reine** Text (Inline-Rich-Text-Tags werden entfernt), sodass der Reader den Satz vorliest, nicht das Markup.

Diese Accessible-Strings werden an die zugrunde liegenden Slate-Widgets angehängt, wenn die Engine `WITH_ACCESSIBILITY` gebaut ist. Auf Builds ohne einkompilierte Barrierefreiheit existieren die Helfer weiterhin und geben denselben `FText` zurück, aber es wird nichts angesagt.

{% hint style="info" %}
Diese Unterstützung ist **ausgelegt für** die Windows-Screen-Reader (Sprachausgabe/Narrator und NVDA), die Standard-Slate-Accessible-Text vorlesen. Die Engine-Barrierefreiheit variiert je nach Plattform und Engine-Version; behandle jede Plattform außerhalb davon als ungetestet, bis du es in deinem eigenen Build verifiziert hast.
{% endhint %}

> 📸 **Bild-Platzhalter:** `accessibility-screenreader-narrator.png` — Narrator liest eine gesperrte Choice vor.
> *Setup:* PIE läuft mit aktivierter Windows-Sprachausgabe (Narrator), Fokus auf einem gesperrten Choice-Button. Die Narrator-Bildunterschrift/Overlay zeigt den gesprochenen Text „Locked choice 1: Persuade the guard — Requires Persuasion 3". Bildunterschrift: „Gesperrte Choices sagen den Grund an, nicht nur den Zustand."

---

## Input-Remapping (Enhanced Input)

Die Dialog-Steuerung — Zeile weiterschalten, Typewriter/Voice überspringen, Choice auswählen — wird über **Enhanced Input** gesteuert. Das bedeutet, die Bindings liegen im Input-Mapping-Context deines Projekts, sodass Spieler sie über die Tasten-Remapping-UI neu belegen können, die dein Spiel ohnehin schon bietet. MayDialogue codiert weder ein Eingabegerät noch eine feste Taste fest.

Das vollständige Setup, die beteiligten Input Actions und wie das Plugin sie konsumiert, behandelt der Runtime-Guide:

[→ Enhanced-Input-Integration](../runtime/enhanced-input.md)

---

## API-Referenz

Alle Helfer liegen im Header `MayDialogueAccessibility.h` im Namespace `MayDialogue::Accessibility`. Es sind reine `FText`/`FString`/`float`-Helfer ohne UObject- oder UMG-Abhängigkeit, sodass ein eigenes Slate- **oder** UMG-Widget den Header inkludieren und sie wiederverwenden kann.

| Symbol | Signatur | Zweck |
|---|---|---|
| `ClampTextScale` | `float ClampTextScale(float InScale)` | Begrenzt `DialogueTextScale` auf `[0.5, 3.0]`. Zentralisiert die Grenzen, sodass beide Schichten übereinstimmen. |
| `GetColorblindChoicePrefix` | `FString GetColorblindChoicePrefix(bool bEnabled, bool bIsAvailable)` | Gibt den 🔒-Schloss-Glyph-Präfix zurück, wenn die Einstellung an ist **und** die Choice nicht verfügbar ist; sonst leerer String. |
| `MakeSpeakerAccessibleText` | `FText MakeSpeakerAccessibleText(const FText& SpeakerName)` | Baut `Speaker: <Name>`. Gibt für einen leeren Namen leeren `FText` zurück. |
| `MakeDialogueTextAccessibleText` | `FText MakeDialogueTextAccessibleText(const FText& MessageText)` | Gibt den reinen Dialogsatz für den Reader zurück. |
| `MakeChoiceAccessibleText` | `FText MakeChoiceAccessibleText(int32 ChoiceIndex, const FText& ChoiceText, bool bIsAvailable, const FText& UnavailableReason)` | Baut die Pro-Choice-Ansage. `ChoiceIndex` ist 0-basiert, wird aber 1-basiert gedruckt. Stellt „Locked choice" voran und hängt den Grund an, wenn nicht verfügbar. |
| `LockGlyph` | `constexpr TCHAR LockGlyph[]` | Das Unicode-Schloss-Glyph (`U+1F512`) plus ein nachfolgendes Leerzeichen, verwendet als farbenblind-Präfix. |

### Settings, die diese Helfer lesen

| Setting | Typ | Default | Wo |
|---|---|---|---|
| `DialogueTextScale` | `float` | `1.0` (begrenzt `0,5–3,0`) | Project Settings → Plugins → MayDialogue → Accessibility |
| `bColorblindSafeChoiceCues` | `bool` | `true` | Project Settings → Plugins → MayDialogue → Accessibility |

Beide Felder sind Teil von `UMayDialogueSettings` — siehe die vollständige [Projekt-Einstellungen-Referenz](../reference/project-settings.md).

---

## Edge-Cases & Stolperfallen

- **Der Scale wird zur Anzeigezeit gelesen, nicht beim Dialog-Start.** Eine Änderung von `DialogueTextScale` mitten in PIE greift ab der nächsten angezeigten Zeile; sie skaliert eine bereits sichtbare Zeile nicht nachträglich.
- **Werte außerhalb des Bereichs werden still begrenzt.** `0.2` wird `0.5`, `5.0` wird `3.0`. Es gibt keine Warnung — `ClampTextScale` bringt den Wert einfach in den Bereich.
- **`bColorblindSafeChoiceCues` markiert nur *sichtbare* gesperrte Choices.** Eine Choice, deren Requirement `FailedAndHidden` zurückgibt, wird gar nicht gerendert und erhält daher kein Glyph — es gibt nichts zu markieren.
- **Das Schloss-Glyph ist reiner Text in der Beschriftung.** Wenn du die Choice-Textbreite für das Layout selbst misst, berücksichtige das zusätzliche Glyph + Leerzeichen. Die Standard-Widgets tun das bereits.
- **Eigene Widgets müssen sich anmelden.** Wenn du ein UMG-Widget geschrieben hast, das **nicht** von den Standard-Komponenten-Widgets erbt, ist nichts davon automatisch — rufe die Helfer selbst auf: `ClampTextScale` für deine Schriftgrößen, `GetColorblindChoicePrefix` beim Bauen von Choice-Beschriftungen und schiebe die `Make*AccessibleText`-Strings über `SetAccessibleBehavior(EAccessibleBehavior::Custom, ...)` auf deine Widgets (derselbe Aufruf, den beide eingebauten Schichten unter `WITH_ACCESSIBILITY` nutzen).
- **Screen-Reader-Ausgabe braucht `WITH_ACCESSIBILITY`.** Shipping-Builds, die Barrierefreiheit herauskompilieren, sagen nichts an, obwohl der Helfer-Text weiterhin erzeugt wird.

---

## Siehe auch

- [Projekt-Einstellungen (Referenz)](../reference/project-settings.md) — jedes `UMayDialogueSettings`-Feld, inklusive beider Barrierefreiheits-Schalter.
- [Enhanced-Input-Integration](../runtime/enhanced-input.md) — remappbare Dialog-Steuerung.
- [UI → Choice List & Choice Button](../ui/choice-list.md) — wo der farbenblind-Präfix und der Accessible-Choice-Text angewendet werden.
- [UI → Slate-Debug-Widget](../ui/slate-debug-widget.md) — das immer aktive Standard-Widget, das alles oben Genannte berücksichtigt.
