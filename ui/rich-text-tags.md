# Rich-Text-Tags

MayDialogue liefert **vier Rich-Text-Decorators**, die Inline-Formatierung im Dialog-Text erlauben. Sie arbeiten mit UE's `URichTextBlock`.

## Die vier Decorators

| Klasse | Tag | Wirkung |
| --- | --- | --- |
| `UMayDialogueShakeDecorator` | `<shake>Text</shake>` | Per-Zeichen zufälliges Offset-Jitter. |
| `UMayDialogueWaveDecorator` | `<wave>Text</wave>` | Sinusförmiges Auf-und-Ab der Zeichen. |
| `UMayDialogueColorDecorator` | `<color=#RRGGBB>Text</color>` oder `<color=red>Text</color>` | Farbwechsel. |
| `UMayDialogueBoldDecorator` | `<b>Text</b>` | Fett. |

Alle Decorators sind **funktional und einsatzbereit**.

## Registrierung

Im Designer deines Text-Widgets: auf den `URichTextBlock` → **DecoratorClasses** → **Add Element** → die Decorator-Klassen hinzufügen.

Für das **Slate-Debug-Widget** sind sie bereits registriert – das nutzt die Factory-Funktionen (`MakeBoldDecorator()`, `MakeColorDecorator()`, etc.) direkt.

## Beispiel

```
Hallo <color=#FF4444>Welt</color>!
Dieser Text <shake>zittert</shake>,
und dieser <wave>welt sanft</wave>.
<b>Wichtig</b>: Das alles ist gleichzeitig möglich.
```

Ergebnis:

* *„Hallo"* in Standard-Farbe.
* *„Welt"* in Rot.
* *„zittert"* mit Per-Character-Shake.
* *„welt sanft"* mit Wellen-Animation.
* *„Wichtig"* in Fett.

## Color-Syntax

| Form | Beispiel | Wirkung |
| --- | --- | --- |
| Hex mit `#` | `<color=#FF0000>` | Rot. |
| Hex ohne `#` | `<color=FF0000>` | Funktioniert auch. |
| Named Color | `<color=red>` | Basis-Farbnamen (`red`, `green`, `blue`, `yellow`, `white`, `black` etc.) |

## Nestable

Decorators **verschachteln sich**:

```
<color=yellow><b>Wichtig</b></color>
```

ergibt gelben Fett-Text.

```
<shake><color=red>!!!</color></shake>
```

ergibt rotes, zitterndes *!!!*.

## Advance-Handler

Die Decorators rendern kontinuierlich auch **während** der Typewriter durch den Text läuft. Das heißt: Sobald die ersten Zeichen von `<shake>X</shake>` enthüllt sind, beginnt der Shake am bereits sichtbaren Teil.

## Vermischen mit Control-Tags

Control-Tags (`<pause>`, `<speed>`) und visuelle Tags können frei gemischt werden:

```
Du bist tot. <pause=1> <color=red><shake>Tot.</shake></color>
```

Der Parser konsumiert `<pause=1>`, die visuellen Tags bleiben intakt, der RichTextBlock rendert sie während des Typewriters.

## Performance

* Shake / Wave aktualisieren ihre Position im RichTextBlock-Tick. Bei sehr langen Shake-Strings kann das leicht Performance kosten – aber in praktischen Dialogen kein Problem.
* Color und Bold sind reine Layout-Settings und kostenlos.

## Einschränkungen & Roadmap

* Keine Animations-Tags (`<fade>`, `<flash>`, …). Wenn du das brauchst: eigener Decorator als Blueprint oder C++.
* Keine Audio-Tags (`<sfx>`). Für Inline-Sounds: `Fire Event` oder `Play Sound`-Action-Node im Graph.
