# Troubleshooting

Wenn etwas nicht funktioniert, findest du hier Antworten.

## Struktur

* [Häufige Probleme](common-issues.md) – die wichtigsten *„Warum geht mein Dialog nicht?"*-Antworten.
* [Debug-Tipps](debugging-tips.md) – Checkliste + Werkzeuge für systematisches Debugging.
* [Bekannte Issues](known-issues.md) – Stand der offenen Bugs und Roadmap-Items.

## Empfohlener Debug-Flow

```mermaid
flowchart TD
    A[Dialog läuft nicht wie erwartet]
    A --> B{Lief er im Preview-Runner?}
    B -->|Ja| C{Im PIE gehts nicht}
    B -->|Nein| D[Text/Struktur-Fehler]
    C --> E[Participant/Tag-Problem]
    C --> F[Widget-Setup-Problem]
    D --> G[Validator-Checks]
    E --> H[Debugger + Watch]
    F --> I[Widget-Bindings prüfen]
    G --> J{Fix}
    H --> J
    I --> J
```

## Schnelle Diagnose-Fragen

* **Startet der Dialog gar nicht?** → [common-issues.md#dialog-start](common-issues.md)
* **Kein Widget erscheint?** → [common-issues.md#widget](common-issues.md)
* **Choices fehlen?** → [common-issues.md#choices](common-issues.md)
* **Audio läuft nicht?** → [common-issues.md#audio](common-issues.md)
* **Variable wird nicht übernommen?** → [common-issues.md#variables](common-issues.md)
* **Crash oder Log-Error?** → [debugging-tips.md](debugging-tips.md)
