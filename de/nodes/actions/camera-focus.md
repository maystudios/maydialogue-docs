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

## Auflösungs-Reihenfolge

Ein einzelner Camera-Focus-Node unterstützt vier Konfigurationen. Die Runtime wertet sie von oben nach unten aus und nutzt die erste, die gesetzt ist:

| Priorität | Aktive Property | Resultat |
|---|---|---|
| 1 | `CameraSequence` | Eine Level Sequence übernimmt die Kamera (Cine Camera + Camera Cuts Track). |
| 2 | `CameraAnchorTag` | Findet einen `AMayDialogueCameraAnchor`-Actor im Level per Tag und macht ihn per `SetViewTargetWithBlend` zum neuen ViewTarget. |
| 3 | `FocusSpeakerTag` + `ShotTag` | Komponiert `ShotAnchors[ShotTag]` des Participants mit der Actor-Transform, spawnt eine transiente `CineCameraActor` dort und macht sie zum ViewTarget. |
| 4 | `FocusSpeakerTag` allein | Klassischer Pfad: dreht die `ControlRotation` des Spielers zum Sprecher (kein ViewTarget-Wechsel). |

Niedriger priorisierte Pfade werden übersprungen, sobald ein höherer greift. Das ursprüngliche ViewTarget wird beim ersten Switch gecached und bei Dialog-Ende automatisch wiederhergestellt (Blend-Zeit = `DefaultAnchorRestoreBlendTime` aus den Project Settings).

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `FocusSpeakerTag` | `FGameplayTag` (`Dialogue.Speaker.*`) | Der Participant, auf den sich Pfad 3 und 4 beziehen. |
| `BlendTime` | `float` | Blend-Dauer in Sekunden. `-1` = Wert aus den Projekt-Settings (`DefaultCameraBlendTime`). |
| `CameraOffset` | `FVector` | Zusätzlicher Welt-Offset (nur Pfad 4 — rotate-controller). |
| `FOVOverride` | `float` | FOV in Grad während des Focus (Pfad 3 + 4). `0` = kein Override. Wird bei Dialog-Ende zurückgesetzt. |
| `bShowDialogueText` | `bool` | Zeigt Dialogtext und wartet auf Spieler-Advance (verhält sich wie SayLine). Orthogonal zum Kamera-Pfad. |
| `DialogueText` | `FText` | Text, der gezeigt wird wenn `bShowDialogueText = true`. |
| `CameraSequence` | `TSoftObjectPtr<ULevelSequence>` | Pfad 1 — Level Sequence übernimmt die Kamera. |
| `bWaitForSequenceEnd` | `bool` | Pfad 1 — Dialog wartet auf `OnFinished` der Sequence. |
| `CameraAnchorTag` | `FGameplayTag` (`Dialogue.CameraAnchor.*`) | Pfad 2 — Tag des im Level platzierten Anchor-Actors. |
| `ShotTag` | `FGameplayTag` (`Dialogue.Shot.*`) | Pfad 3 — Schlüssel in `Participant->ShotAnchors`. Erfordert `FocusSpeakerTag`. |

## Camera Anchor Actor

`AMayDialogueCameraAnchor` ist ein Actor, der von `ACineCameraActor` erbt. Platziere ihn im Level dort, wo die Kamera während eines Shots stehen soll — der Editor zeigt das CineCamera-Frustum, Brennweite, Blende, Schärfentiefe etc. live im Viewport.

| Property | Zweck |
|---|---|
| `AnchorTag` | Identifier (sollte pro Level eindeutig sein). Wird vom Camera-Focus-Node über `CameraAnchorTag` referenziert. |
| `BlendTimeOverride` | Pro-Anchor Override. Negativer Wert = Node-`BlendTime` bzw. Project-Default. |

Workflow:

1. Einen `MayDialogueCameraAnchor`-Actor ins Level setzen, wo die Kamera stehen soll.
2. Shot im Viewport komponieren (Position, Rotation, Brennweite, Blende).
3. `AnchorTag` setzen (z. B. `Dialogue.CameraAnchor.MerchantHall.OverShoulder`).
4. Im Dialogue-Asset im Camera-Focus-Node `CameraAnchorTag` auf denselben Tag setzen.

## Shot Anchors am Participant

`UMayDialogueParticipant::ShotAnchors` ist eine `TMap<FGameplayTag, FTransform>`, die ein kleines „Shot-Vokabular" **pro NPC** definiert (Closeup, OverShoulder, Wide, …). Die Transforms sind **relativ zum Participant-Actor** und bewegen sich mit ihm mit.

Workflow:

1. An der `MayDialogueParticipant`-Komponente des NPCs `ShotAnchors` mit einem Eintrag pro Shot-Typ befüllen (z. B. `Dialogue.Shot.Closeup` → relative Transform).
2. Im Dialogue-Asset im Camera-Focus-Node `FocusSpeakerTag` auf den Sprecher und `ShotTag` auf den Shot setzen.

Zur Laufzeit wird ein transienter `CineCameraActor` an `ShotAnchors[ShotTag] * Participant->GetActorTransform()` platziert und als ViewTarget gesetzt. Der Actor wird über alle Shot-Knoten des Dialogs hinweg wiederverwendet und bei Dialog-Ende automatisch zerstört.

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
- `ShotTag` ohne `FocusSpeakerTag` ist ein **Validator-Fehler** — die Shot-Map kann ohne Sprecher nicht aufgelöst werden.
- Auf einem Dedicated Server (kein PlayerController) sind die Anchor- / Shot-Pfade No-Ops — korrekt, da nur Clients ein ViewTarget haben.
- Mehrere Camera-Focus-Nodes hintereinander: das ursprüngliche ViewTarget wird beim **ersten** Switch gecached, weitere Nodes blenden zwischen Anchors. Beim Dialog-Ende wird zum Original-Target zurückgeblendet.
