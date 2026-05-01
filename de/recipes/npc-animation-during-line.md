---
description: NPC nickt, gestikuliert oder zeigt auf etwas während eine SayLine abgespielt wird – PlayAnimation-Node.
---

# NPC-Animation während Zeile

## Szenario

Ein NPC nickt zustimmend während er eine Aussage macht. Ein anderer zeigt auf eine Karte an der Wand, während er erklärt. Diese Gesten passieren gleichzeitig mit der SayLine – kein extra Dialog-Schritt, kein Blueprint-Code. Der **PlayAnimation**-Node spielt eine Montage auf dem NPC-Actor parallel zum Textflow.

## Was du lernst

- PlayAnimation-Node mit einer Gestik-Montage verknüpfen.
- `bWaitForEnd = false` für parallele Ausführung vs. `true` für sequenzielle.
- Mehrere Animationen in einem Dialog choreografieren.
- `AdvanceMode = AfterAnimation` an einer SayLine nutzen.

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- Montage-Assets vorhanden: `AM_NPC_Nod`, `AM_NPC_PointRight`.
- NPC-Actor hat einen AnimInstance mit aktivem Slot `DefaultSlot`.

## Mini-Graph – Parallel (Geste + Zeile gleichzeitig)

```text
[Entry]
   │
   ▼
[PlayAnimation: NPC – AM_NPC_Nod  WaitForEnd: false]
   │
   ▼
[SayLine: NPC – "Ja, genau so ist es."  AdvanceMode: AfterVoice]
   │
   ▼
[PlayAnimation: NPC – AM_NPC_PointRight  WaitForEnd: false]
   │
   ▼
[SayLine: NPC – "Dort drüben, hinter dem Turm."  AdvanceMode: AfterVoice]
   │
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `npc-animation-during-line-graph-overview.png` — Dialog-Graph mit zwei PlayAnimation-Nodes vor den SayLines.
> *Setup:* Asset `DA_Guide_Explanation` geöffnet. Entry → PlayAnimation "Nod" (orange Box) → SayLine → PlayAnimation "PointRight" (orange Box) → SayLine → Exit. `bWaitForEnd = false` an beiden PlayAnimation-Nodes im Details-Panel sichtbar.

## Schritt-für-Schritt

### 1. PlayAnimation-Node einfügen

Vom Entry-Output → **Create Node → Play Animation**:

| Property | Wert |
|----------|------|
| `TargetParticipantTag` | `Dialogue.Speaker.Guide` |
| `Montage` | `AM_NPC_Nod` |
| `StartSection` | `Default` |
| `bWaitForEnd` | `false` |

`bWaitForEnd = false` → Dialog-Flow geht sofort zur nächsten Node, Montage läuft parallel.

> 📸 **Bild-Platzhalter:** `npc-animation-during-line-anim-details.png` — Details-Panel des PlayAnimation-Nodes.
> *Setup:* PlayAnimation-Node ausgewählt. Details: `TargetParticipantTag = Dialogue.Speaker.Guide`, `Montage = AM_NPC_Nod`, `StartSection = "Default"`, `bWaitForEnd = false (Checkbox leer)`.

### 2. SayLine mit AfterVoice

Vom PlayAnimation-Output → SayLine *„Ja, genau so ist es."*:
- `AdvanceModeOverride = AfterVoice` → Zeile endet wenn Voice fertig ist, nicht wenn Spieler klickt.

So läuft die Geste genau während der Voice-Wiedergabe.

### 3. Zweite Gestik-Animation

Vom SayLine-Output → zweiter PlayAnimation-Node (`AM_NPC_PointRight`, `bWaitForEnd = false`) → zweite SayLine.

### 4. Montage-Slot sicherstellen

Damit PlayAnimation-Node funktioniert, muss der NPC-AnimInstance einen Animation-Slot mit Namen `DefaultSlot` haben. Prüfe im AnimInstance-Blueprint unter *Asset Details → Montage Slots*.

### 5. Compile und PIE testen

Im PIE: Dialog starten. Jede PlayAnimation-Node spielt die Montage auf dem NPC, während die SayLine läuft. Kein Klick nötig zwischen Geste und Zeile.

## bWaitForEnd = true – sequenziell

Wenn die SayLine erst nach der Animation starten soll:

```text
[PlayAnimation: AM_NPC_ThinkingPose  WaitForEnd: true]
   │
   ▼
[SayLine: "Hmm... ich glaube..."]
```

Mit `bWaitForEnd = true` stoppt der Dialog-Flow bei PlayAnimation und wartet, bis die Montage endet. Dann erst kommt die SayLine. Gut für dramatische Pausen.

## AdvanceMode = AfterAnimation an SayLine

Alternativ zu PlayAnimation als eigener Node: `AdvanceModeOverride = AfterAnimation` direkt an der SayLine setzen. Die SayLine wartet auf das Ende einer laufenden Montage. Voraussetzung: eine Montage muss bereits laufen (z.B. gestartet durch vorherigen PlayAnimation-Node).

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Guide_Explanation
   └─ ...
```

> 📸 **Bild-Platzhalter:** `npc-animation-during-line-ingame.png` — PIE-Screenshot: NPC nickt, Dialog-Widget zeigt gleichzeitig die Zeile.
> *Setup:* PIE läuft. Im Viewport: NPC-Charakter in der Nod-Pose. Dialog-Widget unten zeigt `"Ja, genau so ist es."` mit Typewriter-Animation. Beide gleichzeitig sichtbar.

## Variation / Weiter gehen

- Spieler-Animation statt NPC: `TargetParticipantTag = Dialogue.Participant.Player` – der Spieler-Character führt eine Reaktions-Animation aus.
- Jump-Scare-Montage + CameraShake kombinieren → [Jump-Scare mit Camera-Shake](jump-scare-shake.md).
- Animation aus Quest-Variable wählen: vor PlayAnimation ein Branch, der je nach Variable `AM_Friendly` oder `AM_Aggressive` wählt.

## Troubleshooting

**Animation spielt nicht.**
Participant-Tag stimmt nicht mit dem NPC-Actor-Tag überein. NPC-AnimInstance hat den `DefaultSlot` nicht. Montage ist nicht für `DefaultSlot` exportiert.

**Animation endet sofort.**
Montage-Length ist sehr kurz oder `EndSection` im Montage-Asset springt direkt zum Ende. Montage im AnimInstance-Preview prüfen.

**Dialog-Flow stoppt bei PlayAnimation obwohl bWaitForEnd = false.**
`bWaitForEnd` ist im Details-Panel auf `true`. Checkbox-State prüfen.
