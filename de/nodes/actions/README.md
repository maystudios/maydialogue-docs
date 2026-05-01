---
description: Übersicht aller Action-Nodes und wann du welchen wählst.
---

# Action-Nodes

Action-Nodes sind sichtbare, eigenständige Schritte im Dialogue-Graphen. Jeder hat einen Input- und einen Output-Pin, ist per Breakpoint pausierbar und im Outline kommentierbar. Nutze sie, wenn **diese Aktion der Hauptpunkt dieses Schritts** ist.

> **Action-Node oder SideEffect-Sub-Node?**
> Wenn die Aktion den Ablauf strukturell prägt oder auf eigene Weise debuggbar sein soll, nimm den Action-Node als eigene Box. Wenn sie nur nebenbei beim Ausführen einer SayLine passiert (z.B. ein Soundeffekt beim Betreten einer Zeile), hänge sie als SideEffect-Pill an den betreffenden Node.

---

## Alle Action-Nodes auf einen Blick

| Node | Kategorie | Was er macht |
|---|---|---|
| [Camera Focus](camera-focus.md) | Kamera | Blendet die Spielerkamera sanft auf einen Sprecher. |
| [Camera Shake](camera-shake.md) | Kamera | Spielt einen Kamera-Shake auf der Spielerkamera ab. |
| [Play Animation](play-animation.md) | Animation | Spielt eine Montage auf einem Participant — optional mit Warten. |
| [Apply Effect](apply-effect.md) | GAS | Wendet einen `UGameplayEffect` auf Spieler oder Ziel an. |
| [Set Variable](set-variable.md) | Daten | Schreibt eine Dialog-Variable (Bool / Int / Float / String / Tag). |
| [Fire Event](fire-event.md) | Daten | Feuert einen GameplayTag-Event an externe Systeme. |
| [Play Sound](play-sound.md) | Audio | Spielt einen Non-Voice-Sound ab (2D oder 3D). |
| [Add Tag](add-tag.md) | GAS | Setzt einen LooseGameplayTag auf einem ASC. |
| [Remove Tag](remove-tag.md) | GAS | Entfernt einen LooseGameplayTag von einem ASC. |
| [Trigger Cue](trigger-cue.md) | GAS | Feuert einen GameplayCue one-shot (Partikel, SFX, UI-Flash). |

---

## Gemeinsame Regeln

- Alle Action-Nodes haben **einen Input- und einen Output-Pin**.
- Die meisten sind **Pass-through** — der Dialog läuft sofort weiter nach Ausführung.
- Ausnahme: `Play Animation` mit `bWaitForMontageEnd = true` pausiert den Dialog bis die Montage endet. `Camera Focus` mit `bWaitForSequenceEnd = true` wartet auf die Level Sequence.
- Breakpoints auf Action-Nodes pausieren die Ausführung vor dem Node — nützlich beim Debuggen komplexer Abläufe.
