---
description: Kamera sanft auf einen Sprecher schwenken — mit oder ohne LevelSequence.
---

# Camera Focus

Blendet die Spielerkamera auf einen bestimmten Participant (Sprecher oder NPC). Du kannst einen manuellen Blend mit Offset und FOV-Override nutzen oder eine LevelSequence für cineastische Kamerafahrten einsetzen.

## Wann nutzen

- **Dramatischer Enthüllungs-Moment** — Kamera schwenkt zum Monster direkt bevor es spricht.
- **Verhör-Szene** — Fokus wechselt je Dialogzeile von NPC zu Spieler und zurück.
- **Cineastischer Monolog** — LevelSequence fährt eine Close-Up-Kamerafahrt am Erzähler entlang.
- **Horror-Schreckmoment** — FOV-Override auf 50° + schneller Blend (0.1 s) für Panik-Gefühl.

---

> 📸 **Bild-Platzhalter:** `camera-focus-node.png` — Node "Camera Focus" im MayDialogue-Graphen.
> *Setup:* Asset im Editor öffnen. Node allein sichtbar mit sauberem Input-Pin links, Output-Pin rechts. Node-Farbe blau-grau (Kamera-Kategorie). Beschriftung der Title-Bar: "Camera Focus".

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `FocusSpeakerTag` | `FGameplayTag` | Tag des Participants, auf den die Kamera schwenkt. Muss unter `Dialogue.Speaker.*` liegen. |
| `BlendTime` | `float` | Blend-Dauer in Sekunden. `-1` = Wert aus den Projekt-Settings (`DefaultCameraBlendTime`). |
| `CameraOffset` | `FVector` | Zusätzlicher Offset relativ zur Ziel-Position des Participants. |
| `FOVOverride` | `float` | FOV in Grad. `0` = kein Override. Wird bei Dialog-Ende zurückgesetzt. |
| `bShowDialogueText` | `bool` | Zeigt Dialogtext und wartet auf Spieler-Advance (verhält sich wie SayLine). |
| `DialogueText` | `FText` | Text, der gezeigt wird wenn `bShowDialogueText = true`. |
| `CameraSequence` | `TSoftObjectPtr<ULevelSequence>` | Optionale LevelSequence anstelle des manuellen Blends. |
| `bWaitForSequenceEnd` | `bool` | Dialog wartet auf Ende der Sequence. Nur aktiv wenn `CameraSequence` gesetzt. |

---

> 📸 **Bild-Platzhalter:** `camera-focus-details.png` — Details-Panel des Camera-Focus-Nodes mit gefüllten Werten.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `FocusSpeakerTag = Dialogue.Speaker.Guard`, `BlendTime = 0.5`, `CameraOffset = (0, 0, 10)`, `FOVOverride = 0`, `bShowDialogueText = false`, `CameraSequence = leer`.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn der Kamera-Schwenk der **zentrale dramatische Schritt** dieses Graph-Abschnitts ist (der Spieler soll bewusst wahrnehmen, dass die Kamera wechselt), nimm den Action-Node. Wenn der Schwenk nur nebenbei beim Betreten einer SayLine passiert, hänge ihn als SideEffect-Pill an die SayLine.

---

## Beispiel: Fokus-Wechsel im Dialog

```text
[SayLine: Wächter "Halt! Wer bist du?"]
  │
  ▼
[CameraFocus: FocusSpeakerTag=Dialogue.Speaker.Player, BlendTime=0.4]
  │
  ▼
[SayLine: Spieler "Ein Freund des Königs."]
  │
  ▼
[CameraFocus: FocusSpeakerTag=Dialogue.Speaker.Guard, BlendTime=0.4]
  │
  ▼
[SayLine: Wächter "Dann passiere."]
```

> 📸 **Bild-Platzhalter:** `camera-focus-example-graph.png` — Graphausschnitt des obigen Beispiels.
> *Setup:* Vier Nodes von links nach rechts: SayLine (Guard) → CameraFocus (Player) → SayLine (Player) → CameraFocus (Guard) → SayLine (Guard). Pins verbunden. Beide CameraFocus-Nodes sichtbar mit `FocusSpeakerTag` im Subtitle.

> 📸 **Bild-Platzhalter:** `camera-focus-ingame-before-after.png` — Split-Screen vor und nach dem Schwenk im PIE-Viewport.
> *Setup:* Links: Spieler-Perspektive auf den Wächter (Guard zentriert). Rechts: nach CameraFocus auf Player — Kamera zeigt Spieler-Gesicht. Beide Bilder mit HUD-Overlay sichtbar.

---

## Fallstricke

{% hint style="warning" %}
**FOV-Override stackt nicht.** Der ursprüngliche FOV wird einmalig beim ersten Override gespeichert. Mehrere CameraFocus-Nodes mit unterschiedlichem `FOVOverride` hintereinander führen nicht zu korrekt verschachteltem Restore. Nutze `FOVOverride` nur einmal pro Dialog oder setze ihn vor Dialog-Ende explizit zurück.
{% endhint %}

{% hint style="info" %}
**AutoFocusSpeaker**: In den Projekt-Settings gibt es `AutoFocusSpeaker`. Wenn aktiv, schwenkt die Kamera automatisch auf den aktuellen Sprecher einer SayLine — dann brauchst du diesen Node nur noch für manuelle Overrides oder LevelSequences.
{% endhint %}

- `CameraSequence` läuft parallel — der Dialog advanced sofort weiter (es sei denn `bWaitForSequenceEnd = true`).
- `bShowDialogueText` und `bWaitForSequenceEnd = true` gleichzeitig: `bShowDialogueText` gewinnt (manueller Advance).
- `FocusSpeakerTag` muss einem registrierten Participant entsprechen — sonst kein Blend, Log-Warning.
