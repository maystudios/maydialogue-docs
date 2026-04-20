# Emotionen & Tag-Container

MayDialogue nutzt **GameplayTagContainer** statt einfacher Enum-Felder, um Emotionen, Intensitäten, Szenen-Kontexte und Stil-Marker zu modellieren. Dieses Kapitel zeigt, warum – und wie du Tags optimal einsetzt.

## Das Datenmodell

Jeder SayLine-Node trägt ein `FGameplayTagContainer EmotionTags`. Beispiele für sinnvolle Tag-Hierarchien:

```
Dialogue.Emotion.Scared
Dialogue.Emotion.Angry
Dialogue.Emotion.Whisper
Dialogue.Intensity.Low
Dialogue.Intensity.High
Dialogue.Scene.Kitchen
Dialogue.Scene.CellarWithMonster
Dialogue.Style.InnerMonologue
```

Ein SayLine kann **mehrere** Tags gleichzeitig tragen:

```
Tags: {
  Dialogue.Emotion.Scared,
  Dialogue.Intensity.High,
  Dialogue.Scene.CellarWithMonster
}
```

## Warum Tag-Container statt Enum?

| Aspekt | Tag-Container | Einfaches Enum |
| --- | --- | --- |
| Mehrere Werte gleichzeitig | ✅ | ❌ |
| Hierarchisch (Parent-Match) | ✅ | ❌ |
| Nachträglich erweiterbar | ✅ | ❌ (Schema-Change) |
| Auto-Complete im Editor | ✅ | ✅ |
| Cross-Projekt-Recycling | ✅ | ❌ |

Der wichtigste Vorteil ist **Hierarchie-Matching**. Ein UI-Widget kann einstellen: *„Zeige rote Umrandung, wenn die aktuelle SayLine irgendein `Dialogue.Emotion`-Tag trägt"* – ohne jeden einzelnen Emotion-Tag zu kennen:

```cpp
if (Message.EmotionTags.HasTag(FGameplayTag::RequestGameplayTag("Dialogue.Emotion")))
{
    // Vereinfacht: irgendeine Emotion ist gesetzt
}
```

Das ist robust gegen neue Tags – eine später hinzugefügte `Dialogue.Emotion.Guilty` matcht automatisch.

## Wer wertet die Tags aus?

**UI, Audio und Animation** sind die typischen Abnehmer:

### UI

Das Speaker-Widget erhält `Message.EmotionTags` bei jedem Speaker-Change. Du kannst im Blueprint-Event `OnSpeakerChanged` eine Portrait-Variante auswählen:

```
if Tags contain "Dialogue.Emotion.Scared":
    set Portrait Image to P_Guard_Scared
elif Tags contain "Dialogue.Emotion.Angry":
    set Portrait Image to P_Guard_Angry
else:
    set Portrait Image to P_Guard_Neutral
```

### Audio

Das Babel-System könnte Pitch-Shifts oder SoundClass-Varianten je nach Intensity-Tag anwenden. Oder ein externes Audio-System hört auf `OnMessageReceived` und modifiziert den Mixer.

### Animation

Ein Face-Blend-Controller hört auf Emotion-Tags und morpht die Gesichts-Shapes entsprechend. Ohne dass der Dialog-Node eine direkte Animation triggern muss.

## Tag-Kategorien im Projekt

Empfehlung für deine Tag-Taxonomie (beliebig anpassbar):

| Kategorie | Zweck | Beispiele |
| --- | --- | --- |
| `Dialogue.Emotion.*` | Grundgefühl | Calm, Scared, Angry, Sad, Happy, Whisper |
| `Dialogue.Intensity.*` | Stärke | Low, Medium, High |
| `Dialogue.Scene.*` | Räumlicher Kontext | Kitchen, Office, Graveyard |
| `Dialogue.Style.*` | Erzählstil | InnerMonologue, Narrator, Shout |
| `Dialogue.Mood.*` | Beziehungs-Ton | Friendly, Neutral, Hostile |

## Tag-Definitionen registrieren

Tags können auf zwei Wegen definiert werden:

1. **Per INI-File** — `Config/Tags/DialogueTags.ini` mit `+GameplayTagList=(Tag="Dialogue.Emotion.Scared")`.
2. **Nativ per C++** — in einer `FNativeGameplayTag` (siehe `VHSGameplayTags.h` als Projekt-Beispiel).

Native Tags haben den Vorteil, dass sie typisierte Referenzen im Code erlauben:

```cpp
// Header
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Dialogue_Emotion_Scared);

// Cpp
UE_DEFINE_GAMEPLAY_TAG_COMMENT(TAG_Dialogue_Emotion_Scared, "Dialogue.Emotion.Scared", "NPC ist erschrocken.");
```

## Tags in Requirements / SideEffects

Das **GAS-Modul** liefert ein `HasTag`-Requirement. Das prüft aber den **ASC-Tag-Container** des Target-Actors, **nicht** den SayLine-Emotion-Container.

Wenn du Logik bauen willst, die „*reagiere, wenn die aktuelle SayLine einen Tag X trägt"* abbildet, gehst du über einen **Event-Listener** am Subsystem:

```cpp
Subsystem->OnMessageReceived.AddDynamic(this, &AMyListener::HandleMessage);

void AMyListener::HandleMessage(const FMayDialogueMessage& Message)
{
    if (Message.EmotionTags.HasTag(TAG_Dialogue_Intensity_High))
    {
        // Spiele Jump-Scare-Audio, triggere Screen-Shake, etc.
    }
}
```

## Tags auf Choices

Zusätzlich zu SayLine-EmotionTags können **Choices** ihre eigenen Tags tragen (`ChoiceTags`):

```
Choice 1: "Ich kenne das Passwort."
  Tags: { Dialogue.Choice.Assertive, Dialogue.Choice.Secret }

Choice 2: "Hallo."
  Tags: { Dialogue.Choice.Friendly }
```

Das **Achievement-System** oder ein **Analytics-Logger** kann auf `OnChoiceMade` hören und die Tags der gewählten Choice auswerten:

```cpp
if (Choice.ChoiceTags.HasTag(TAG_Dialogue_Choice_Assertive))
{
    Achievements->Grant("AssertivePlayer");
}
```

## Zusammengefasst

* Tag-Container statt Enum – hierarchisch, mehrfach, erweiterbar.
* **EmotionTags** auf SayLines, **ChoiceTags** auf Choices.
* UI / Audio / Animation werten Tags als Hooks aus.
* Tag-Subscription per `OnMessageReceived` / `OnChoiceMade`.

Weiter: das Erbe von Epic — [Verhältnis zu CommonConversation](common-conversation.md).
