---
description: Emotion-Tags an SayLines setzen und was UI, Audio und Animation damit machen.
---

# Emotionen & Tags

MayDialogue nutzt **GameplayTag-Container** statt einfacher Enum-Felder, um Emotionen, Intensitäten und Szenen-Kontext an Sprechzeilen zu markieren. Dieses Kapitel zeigt, wie du Tags setzt und wie UI, Audio und Animation darauf reagieren.

## Was sind Emotion-Tags?

Jeder SayLine-Node trägt ein `EmotionTags`-Feld — einen `FGameplayTagContainer`, in den du beliebig viele Tags einträgst. Diese Tags sind **kein Enum** — sie haben eine Hierarchie und können kombiniert werden.

Beispiel-Tags:

```text
Dialogue.Emotion.Scared
Dialogue.Emotion.Angry
Dialogue.Emotion.Whisper
Dialogue.Intensity.Low
Dialogue.Intensity.High
Dialogue.Scene.CellarWithMonster
Dialogue.Style.InnerMonologue
```

Eine SayLine kann mehrere Tags gleichzeitig tragen:

```text
Tags: { Dialogue.Emotion.Scared, Dialogue.Intensity.High, Dialogue.Scene.CellarWithMonster }
```

> 📸 **Bild-Platzhalter:** `sayline-emotion-tags-details.png` — Details-Panel einer SayLine mit gesetzten EmotionTags.
> *Setup:* SayLine-Node im Graph ausgewählt. Details-Panel rechts zeigt unter „Emotion Tags": drei Einträge: `Dialogue.Emotion.Scared`, `Dialogue.Intensity.High`, `Dialogue.Scene.CellarWithMonster`. Daneben im Graph-Node: die drei Tags als blaue Chips unter dem Inline-Text sichtbar.

## Warum Tag-Container statt Enum?

| Aspekt | Tag-Container | Einfaches Enum |
| --- | --- | --- |
| Mehrere Werte gleichzeitig | Ja | Nein |
| Hierarchisch matchbar | Ja | Nein |
| Nachträglich erweiterbar | Ja, ohne Schema-Änderung | Nein |
| Auto-Complete im Editor | Ja | Ja |

Der wichtigste Vorteil ist **Hierarchie-Matching**. Ein Widget kann prüfen: „Hat diese SayLine irgendeinen Emotion-Tag?" — ohne jeden einzelnen Tag zu kennen:

```cpp
if (Message.EmotionTags.HasTag(FGameplayTag::RequestGameplayTag("Dialogue.Emotion")))
{
    // Irgendeinen Emotions-Tag gefunden — zum Beispiel für Portrait-Wechsel
}
```

Eine neu hinzugefügte `Dialogue.Emotion.Guilty` matcht automatisch, ohne Code-Änderung.

## Tags an einer SayLine setzen

**Im Graph:**

> 📸 **Bild-Platzhalter:** `sayline-add-emotion-tag.png` — Emotion-Tag zu einer SayLine hinzufügen.
> *Setup:* SayLine-Node im Graph ausgewählt. Details-Panel, Feld `EmotionTags`, `+`-Button angeklickt. Dropdown mit Tag-Picker öffnet sich. Eingetragen: `Dialogue.Emotion.Scared`. Der Tag erscheint danach als blauer Chip im Node-Body.

Tags können direkt im Graph-Node als farbige Chips gelesen werden — ohne Details-Panel zu öffnen.

## Wer reagiert auf Tags?

### UI: Portrait-Varianten

Das Widget erhält `Message.EmotionTags` bei jeder neuen SayLine. Im Blueprint-Event `OnMessageReceived` kannst du das Portrait wechseln:

> 📸 **Bild-Platzhalter:** `bp-portrait-switch-by-tag.png` — Blueprint-Graph im Widget: Portrait anhand von Emotion-Tags wechseln.
> *Setup:* Widget-Blueprint, Event `On Message Received`. `Message.EmotionTags` → `HasTag (Dialogue.Emotion.Scared)` → Branch. True-Zweig: `Set Brush from Texture (TX_Guard_Scared)`. False-Zweig: weiterer Branch mit `HasTag (Dialogue.Emotion.Angry)` → `TX_Guard_Angry` / `TX_Guard_Neutral`. Alle Nodes beschriftet, Pins sichtbar.

```cpp
void UMyDialogueWidget::OnMessageReceived(const FMayDialogueMessage& Message)
{
    if (Message.EmotionTags.HasTag(TAG_Emotion_Scared))
        PortraitImage->SetBrushFromTexture(TX_Guard_Scared);
    else if (Message.EmotionTags.HasTag(TAG_Emotion_Angry))
        PortraitImage->SetBrushFromTexture(TX_Guard_Angry);
    else
        PortraitImage->SetBrushFromTexture(TX_Guard_Neutral);
}
```

### Audio: Intensität und Stimmung

Ein Audio-Manager hört auf `OnMessageReceived` und passt den Mixer oder das Babel-Profil an:

```cpp
void UAudioManager::HandleMessage(const FMayDialogueMessage& Message)
{
    if (Message.EmotionTags.HasTag(TAG_Intensity_High))
        SetMixerSnapshot("DialogueTense");
    else
        SetMixerSnapshot("DialogueCalm");
}
```

### Animation: Gesichtsblending

Ein Face-Blend-Controller kann Morph-Targets anhand der Emotion-Tags steuern — ohne dass der Dialog-Node eine direkte Animations-Abhängigkeit hat. Der Controller hört auf `OnMessageReceived` und blended unabhängig vom Dialog.

> 📸 **Bild-Platzhalter:** `ingame-emotion-portrait.png` — In-Game-Dialog mit gewechseltem Portrait.
> *Setup:* PIE-Modus, aktiver Dialog. Das Widget zeigt das ängstliche Guard-Portrait (TX_Guard_Scared) mit rotem Tinting. Darunter der Text „D-da ist... etwas im Keller." (Typewriter läuft). Keine Choices sichtbar. Hintergrund: Keller-Level.

## Tags auf Choices

Choices können zusätzlich eigene `ChoiceTags` tragen:

```text
Choice 0: "Ich kenne das Passwort."
  Tags: { Dialogue.Choice.Assertive, Dialogue.Choice.Secret }

Choice 1: "Hallo."
  Tags: { Dialogue.Choice.Friendly }
```

`OnChoiceMade` übergibt die Tags der gewählten Choice — nützlich für Achievements oder Analytics:

```cpp
void AMyTracker::OnChoiceMade(int32 Index, const FMayDialogueChoiceEntry& Choice)
{
    if (Choice.ChoiceTags.HasTag(TAG_Choice_Assertive))
        Achievements->Grant("AssertivePlayer");
}
```

## Eigene Tag-Taxonomie anlegen

Tags werden per INI oder nativ in C++ registriert.

**Per INI** (`Config/Tags/DialogueTags.ini`):

```ini
[/Script/GameplayTags.GameplayTagsList]
+GameplayTagList=(Tag="Dialogue.Emotion.Scared",DevComment="NPC ist verängstigt.")
+GameplayTagList=(Tag="Dialogue.Intensity.High",DevComment="Intensive Szene.")
```

**Nativ (C++):**

```cpp
// Header
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Dialogue_Emotion_Scared)

// Cpp
UE_DEFINE_GAMEPLAY_TAG_COMMENT(TAG_Dialogue_Emotion_Scared,
    "Dialogue.Emotion.Scared", "NPC ist verängstigt.")
```

Native Tags haben den Vorteil typisierter Referenzen im Code — kein String-Typo möglich.

## Empfohlene Tag-Kategorien

| Kategorie | Zweck | Beispiele |
| --- | --- | --- |
| `Dialogue.Emotion.*` | Grundgefühl des Sprechers | Calm, Scared, Angry, Sad, Happy, Whisper |
| `Dialogue.Intensity.*` | Stärke der Emotion | Low, Medium, High |
| `Dialogue.Scene.*` | Räumlicher Kontext | Kitchen, Office, Graveyard, CellarWithMonster |
| `Dialogue.Style.*` | Erzählstil | InnerMonologue, Narrator, Shout |
| `Dialogue.Mood.*` | Beziehungston | Friendly, Neutral, Hostile |
| `Dialogue.Choice.*` | Markierung gewählter Antworten | Assertive, Friendly, Aggressive |

{% hint style="info" %}
**Eigene Tag-Kategorien** lassen sich jederzeit ergänzen — einfach im INI oder per `UE_DEFINE_GAMEPLAY_TAG_COMMENT` registrieren. Vorhandener Code matcht automatisch über Hierarchie, ohne Änderungen.
{% endhint %}

> 📸 **Bild-Platzhalter:** `tag-picker-in-editor.png` — Tag-Picker im Dialog-Asset mit eigenen Dialogue-Tags.
> *Setup:* SayLine-Node ausgewählt, Tag-Picker-Dropdown für `EmotionTags` geöffnet. Sichtbar: Baum-Struktur mit `Dialogue > Emotion` aufgeklappt: `Calm`, `Scared`, `Angry`, `Sad`, `Whisper`. Darunter `Dialogue > Intensity`: `Low`, `Medium`, `High`. Ein Tag ist gerade per Hover markiert.

## Zusammenfassung

- Emotion-Tags sind `GameplayTagContainer` — hierarchisch, kombinierbar, erweiterbar ohne Schema-Änderung.
- Tags setzen: im Details-Panel der SayLine oder direkt per Doppelklick.
- Abnehmer: UI (Portrait-Varianten), Audio (Mixer/Babel), Animation (Face-Blending).
- Choice-Tags auf `FMayDialogueChoiceEntry` für Achievements und Analytics.
- `OnMessageReceived` und `OnChoiceMade` sind die Hooks für externe Systeme.

Ende der Kern-Konzepte. Weiter mit dem **Editor**: [Asset-Editor](../editor/README.md).
