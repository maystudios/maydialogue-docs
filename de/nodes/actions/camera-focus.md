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

## Auflösungs-Reihenfolge — welcher Pfad läuft

Ein Camera-Focus-Node unterstützt drei Konfigurationen. Die Runtime wertet sie von oben nach unten aus und nutzt die erste, die gesetzt ist:

| Priorität | Aktive Property | Resultat |
|---|---|---|
| 1 | `CameraSequence` | Eine Level Sequence übernimmt die Kamera (Cine Camera + Camera Cuts Track). |
| 2 | `CameraAnchorTag` | Findet einen `AMayDialogueCameraAnchor`-Actor per Tag und macht ihn per `SetViewTargetWithBlend` zum neuen ViewTarget. |
| 3 | `FocusSpeakerTag` allein | Legacy-Pfad: dreht die `ControlRotation` des Spielers smooth zum Sprecher. **Kein ViewTarget-Switch** — die echte Pawn-Kamera (Spring-Arm, Lag, Post-Process) bleibt aktiv. |

Niedriger priorisierte Pfade werden übersprungen, sobald ein höherer greift. Das ursprüngliche ViewTarget wird beim ersten Switch gecached und bei Dialog-Ende automatisch wiederhergestellt (Blend-Zeit = `DefaultAnchorRestoreBlendTime` aus den Project Settings).

## Override-Reihenfolge — wer gewinnt für jeden Wert

Alle Werte folgen demselben Muster: **most-specific wins**, mit Sentinel-Werten als „nicht gesetzt → fall through".

| Wert | Quelle 1 (Priorität) | Quelle 2 | Fallback |
|---|---|---|---|
| **BlendTime (Anchor-Pfad)** | Node `BlendTime` ≥ 0 | Anchor `BlendTimeOverride` ≥ 0 | Settings `DefaultCameraBlendTime` |
| **BlendTime (Legacy-Pfad)** | Node `BlendTime` ≥ 0 | — | Settings `DefaultCameraBlendTime` |
| **Restore-BlendTime** | — | — | Settings `DefaultAnchorRestoreBlendTime` |
| **FOV (Legacy-Pfad)** | Node `FOVOverride` > 0 | — | Aktueller `PlayerCameraManager`-FOV |
| **FOV (Anchor-Pfad)** | — | — | CineCamera-Settings am Anchor (Brennweite, Aperture etc.) |

### Look-at-Position (nur Legacy-Pfad)

Die einzige Stelle mit **additivem** statt override-Verhalten:

```
Look-at = SpeakerActor.Location
          + (Node.bIgnoreParticipantOffset ? 0 : Participant.CameraTargetOffset)
          + Node.CameraOffset
```

`Participant.CameraTargetOffset` setzt der Designer **einmal pro NPC** (z. B. „Kopfhöhe = +60 cm"), `Node.CameraOffset` ist situativ pro Szene. Beides wird sinnvoll addiert statt Doppelarbeit zu erzwingen. `bIgnoreParticipantOffset = true` am Node deaktiviert die Participant-Komponente für Spezialfälle.

Im Anchor-Pfad sind diese Offsets **wirkungslos** — der Anchor-Actor definiert seine Pose komplett selbst.

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `FocusSpeakerTag` | `FGameplayTag` (`Dialogue.Speaker.*`) | Sprecher für Pfad 3. |
| `BlendTime` | `float` | Blend-Dauer in Sekunden. `-1` = Wert aus den Projekt-Settings (`DefaultCameraBlendTime`). |
| `CameraOffset` | `FVector` | Zusätzlicher Welt-Offset (nur Pfad 3, additiv mit `Participant.CameraTargetOffset`). |
| `bIgnoreParticipantOffset` | `bool` | Wenn `true`: ignoriert `Participant.CameraTargetOffset`, nur `CameraOffset` zählt. Default: `false`. |
| `bLockLookInputDuringBlend` | `bool` | Legacy-Pfad: sperrt Spieler-Look-Input während des Blends, damit der Lerp nicht von Maus/Stick überschrieben wird. Auf Anchor-Pfad wirkungslos. Default: `true`. |
| `bLockMoveInputDuringBlend` | `bool` | Legacy-Pfad: sperrt Bewegungs-Input des Spielers während des Focus (kein Walk-and-Talk). Auf Anchor-Pfad wirkungslos. Default: `true`. |
| `FOVOverride` | `float` | FOV in Grad während des Focus (nur Pfad 3). `0` = kein Override. |
| `bShowDialogueText` | `bool` | Zeigt Dialogtext und wartet auf Spieler-Advance (verhält sich wie SayLine). Orthogonal zum Kamera-Pfad. |
| `DialogueText` | `FText` | Text, der gezeigt wird wenn `bShowDialogueText = true`. |
| `CameraSequence` | `TSoftObjectPtr<ULevelSequence>` | Pfad 1 — Level Sequence übernimmt die Kamera. |
| `bWaitForSequenceEnd` | `bool` | Pfad 1 — Dialog wartet auf `OnFinished` der Sequence. |
| `CameraAnchorTag` | `FGameplayTag` (`Dialogue.CameraAnchor.*`) | Pfad 2 — Tag des Anchor-Actors. |

## Camera Anchor Actor — das Kern-Werkzeug für inszenierte Shots

`AMayDialogueCameraAnchor` ist ein Actor, der von `ACineCameraActor` erbt. Platziere ihn im Level — der Editor zeigt das CineCamera-Frustum, Brennweite, Blende, Schärfentiefe etc. live im Viewport. Designer komponieren den Shot per Viewport-Drag, nicht per Vektor-Tippen.

| Property | Zweck |
|---|---|
| `AnchorTag` | Identifier (sollte pro Level eindeutig sein). Wird vom Camera-Focus-Node über `CameraAnchorTag` referenziert. |
| `BlendTimeOverride` | Pro-Anchor Override. Negativer Wert = Node-`BlendTime` bzw. Project-Default. |

### Welt-fester Shot

Anchor lose im Level platzieren — er bleibt an seiner Position, egal wo der NPC steht. Gut für Set-Pieces (Throne Room, Marktstand, Cutscene-Kamerapunkt).

### Charakter-fester Shot — Anchor an den NPC parenten

Soll der Shot **mit dem NPC mitwandern** (Closeup, Over-Shoulder, …):

1. Den `MayDialogueCameraAnchor` ins Level setzen.
2. Im **World Outliner** den Anchor per Drag-and-Drop auf den NPC-Actor ziehen → er wird zum **Child-Actor**, dessen Transform jetzt relativ zum NPC ist.
3. Shot im Viewport komponieren — der Anchor folgt dem NPC automatisch, wo immer er sich bewegt.
4. `AnchorTag` setzen, im Dialogue-Asset referenzieren.

Das ist der komplette Mechanismus für „NPC-relative Shots": natives UE-Actor-Parenting, kein separates Konzept. Vorteil gegenüber unsichtbaren Transform-Maps: **alles im Viewport sichtbar, draggbar, mit Live-CineCamera-Preview**.

Tipp: für NPC-Blueprints lassen sich die Anchors als Child-Actor-Komponenten direkt im Blueprint anlegen — dann liefert jeder NPC-Spawn sein Shot-Vokabular automatisch mit.

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
- Anchor-Existenz wird **nicht** vom Validator geprüft (Anchors leben im Level, der Validator läuft auf dem Asset). Fehlende Anchors loggen zur Laufzeit eine `Warning` und der Node fällt auf den nächsten Pfad zurück.
- Auf einem Dedicated Server (kein PlayerController) ist der Anchor-Pfad ein No-Op — korrekt, da nur Clients ein ViewTarget haben.
- Mehrere Camera-Focus-Nodes hintereinander: das ursprüngliche ViewTarget wird beim **ersten** Switch gecached, weitere Nodes blenden zwischen Anchors. Beim Dialog-Ende wird zum Original-Target zurückgeblendet.
