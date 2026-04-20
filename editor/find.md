# Find-in-Dialogue

Die Suche-Funktion findet **jede Text-, Tag- oder Sprecher-Erwähnung** im gesamten Asset.

## Öffnen

* **Toolbar-Button Find** oder `Ctrl+F`.
* Der **FindResults**-Tab schiebt sich in den Vordergrund.

## Was wird durchsucht?

* **SayLine-Texte** (`DialogueText`).
* **Choice-Texte** (`ChoiceText` auf allen Choices).
* **PlayerChoice-Prompts** (`PromptText`).
* **Sprecher-Namen** (`DisplayName` aus dem Speakers-Panel).
* **Variable-Namen**.
* **GameplayTags** (EmotionTags, ChoiceTags, EventTags, …).
* **Editor-Kommentare** (Comment-Boxes und per-Node Kommentar-Strings).

Case-insensitive Substring-Match.

## Ergebnisliste

Pro Treffer eine Zeile mit:

| Spalte | Bedeutung |
| --- | --- |
| **Match-Typ** | *„SayLine-Text"*, *„Choice-Text"*, *„Speaker"*, *„Tag"*, *„Variable"*, … |
| **Kontext** | Der Treffer im Kontext (gekürzt, der Match ist hervorgehoben). |
| **Node** | Klickbare Referenz – Kamera springt im Graph zum Node. |

## Einsatzfälle

### Texte vereinheitlichen

Suche nach alten Begriff-Varianten und ersetze sie manuell per Click-to-Jump.

### Tag-Nutzung prüfen

Suche nach einem GameplayTag (z.B. `Dialogue.Emotion.Scared`) – die Liste zeigt alle Nodes, die ihn referenzieren.

### Variable auditieren

Suche nach einem Variable-Namen, um zu sehen, welche Nodes sie setzen oder prüfen.

### Sprecher-Audit

Suche nach einem Speaker-DisplayName – du siehst alle Nodes, in denen er spricht.

## Grenzen

* **Nur innerhalb des aktuellen Assets.** Cross-Asset-Suche ist nicht eingebaut; für projektweite Suche nutze den globalen UE-Asset-Finder.
* **Keine Regex.** Nur Substring-Match.
* **Nur Text, keine Property-Werte außer Textfeldern.** Z.B. `CameraFocus.BlendTime = 1.5` wird nicht gefunden.

Weiter: [Preview-Runner →](preview-runner.md).
