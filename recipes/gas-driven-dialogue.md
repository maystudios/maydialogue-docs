# GAS-getriebener Dialog

Dieses Rezept kombiniert die drei GAS-Bausteine des Plugins – **CheckAttribute**, **AddTag/RemoveTag** und **ApplyEffect** – zu einem realistischen Mini-Gespräch, das Welt-Zustand sowohl liest als auch schreibt. Wenn du dieses Rezept durchgearbeitet hast, verstehst du den kompletten GAS-Loop von MayDialogue.

## Szenario

Ein Kräuterweib bietet dem Spieler einen Trank an. Der Dialog läuft so:

1. Sie fragt, ob der Spieler verletzt ist – prüft `Health`-Attribut.
2. Ist Health unter 50 → Heilungs-Option verfügbar.
3. Wählt der Spieler „Heile mich" → ApplyEffect (`GE_SmallHeal`), SetVariable-Counter im Participant, AddTag `Story.Met.Herbwoman`.
4. Verlässt der Spieler die Frau ohne Kauf → RemoveTag `Story.Met.Herbwoman` nicht, denn „kennengelernt" bleibt kennengelernt.
5. Beim zweiten Besuch (Tag vorhanden) → alternative Begrüßungs-Variante.

## Beteiligte Nodes

* [Entry](../nodes/core/entry.md), [Exit](../nodes/core/exit.md)
* [Branch](../nodes/core/branch.md) mit [HasTag-Requirement](../gas/requirements.md#umaydlgrequirement_hastag)
* [SayLine](../nodes/core/say-line.md), [PlayerChoice](../nodes/core/player-choice.md)
* [ApplyEffect-Action](../nodes/actions/apply-effect.md)
* [AddTag-Action](../nodes/actions/add-tag.md)
* [SetVariable-SideEffect](../nodes/actions/set-variable.md)
* [CheckAttribute-Requirement](../gas/requirements.md#umaydlgrequirement_checkattribute)

## Graph-Mock-Up

```
[Entry]
   │
   ▼
[Branch]
 ├─ BP1: HasTag("Story.Met.Herbwoman") → [SayLine: "Schön, dich wiederzusehen."]
 └─ BP2: <default>                     → [SayLine: "Ich kenne dich nicht..."] → [AddTag: Story.Met.Herbwoman]
                                             │
                                             └──────┬───────────────────────────┘
                                                    ▼
                                            [PlayerChoice]
                                               ├─ "Einen Trank bitte." (Req: CheckAttribute Health < 50)
                                               │      └─► [ApplyEffect: GE_SmallHeal] (SideEffect: SetVariable PotionsBought +=1)
                                               │              └─► [SayLine: "Besser?"] → [Exit]
                                               └─ "Nein danke."
                                                       └─► [Exit]
```

## Schritt-für-Schritt

### 1. Tags registrieren

Im Projekt (`DefaultGameplayTags.ini` oder C++ native Tag-Declaration):

```
Story.Met.Herbwoman
Dialogue.Speaker.Herbwoman
```

### 2. Asset anlegen

`DA_Herbwoman_Visit`. Speaker *Kräuterweib* mit Tag `Dialogue.Speaker.Herbwoman`. Participant-Variable `PotionsBought` vom Typ `Int`, Default 0, Scope *Participant*.

### 3. Branch mit Wieder-/Ersttreffen

Branch-Node einfügen. Erster BranchPoint mit [HasTag-Requirement](../gas/requirements.md#umaydlgrequirement_hastag):

* `RequiredTag`: `Story.Met.Herbwoman`
* `bCheckOnInstigator`: `true` (Tag liegt am Spieler)

Zweiter BranchPoint ohne Requirements (Default-Pfad).

### 4. First-Visit-Pfad

Nach der einführenden SayLine einen [AddTag-Action-Node](../nodes/actions/add-tag.md):

| Property | Wert |
| --- | --- |
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `Tag` | `Story.Met.Herbwoman` |
| `bApplyPermanent` | `true` (via Infinite-GE oder LooseTag) |

### 5. PlayerChoice mit Attribut-Gate

Der erste Choice-Eintrag bekommt ein [CheckAttribute-Requirement](../gas/requirements.md#umaydlgrequirement_checkattribute):

| Property | Wert |
| --- | --- |
| `Attribute` | `UVHSAttributeSet::Health` |
| `ComparisonOp` | `<` |
| `ComparisonValue` | `50.0` |
| `FailureResult` | `FailedButVisible` |
| `UnavailableReason` | *„Du scheinst gesund."* |

So sieht der Spieler die Option immer, kann sie aber nur wählen, wenn sein Health unter 50 liegt. Bei Hover erscheint die Begründung.

### 6. ApplyEffect + SideEffect

Vom Heilungs-Choice-Output auf einen [ApplyEffect-Action-Node](../nodes/actions/apply-effect.md):

| Property | Wert |
| --- | --- |
| `EffectClass` | `GE_SmallHeal` |
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `EffectLevel` | `1.0` |
| `bApplyFromInstigator` | `true` |

Am ApplyEffect-Node noch einen SideEffect-Sub-Node *SetVariable*:

```
SetVariable
  Name:  PotionsBought
  Scope: Participant
  Type:  Int
  Op:    Increment
  Value: 1
```

### 7. Compile & Test

Debugger mit PIE öffnen. Am ASC des Spielers siehst du nach dem Trank:

* `Health` steigt entsprechend des GameplayEffects.
* `Story.Met.Herbwoman` liegt auf dem Player-ASC.
* Kräuterweib-Participant hat `PotionsBought = 1`.

## Vollständige ApplyEffect-Mechanik

| Schritt | Was passiert |
| --- | --- |
| 1 | Target-ASC auflösen (via ParticipantTag → Participant → Actor → ASC). |
| 2 | Instigator-ASC als Source setzen (für AttributeCaptureSources). |
| 3 | `MakeOutgoingSpec(EffectClass, Level, Context)`. |
| 4 | `ApplyGameplayEffectSpecToSelf` auf Target-ASC. |
| 5 | Bei Failure (kein ASC, kein Target): TaskResult::Abort oder Skip je nach FailBehavior. |

## Design-Fragen: Loose-Tag vs. GameplayEffect

`AddTag`-Node hat zwei Varianten:

| Modus | Pro | Con |
| --- | --- | --- |
| `LooseGameplayTag` | Einfach, kein GE nötig. | Nicht repliziert, nicht persistiert. Client sieht Tag nicht. |
| `Via GameplayEffect` (Infinite) | Repliziert, persistiert (mit GE-Handles), kann gezielt entfernt werden. | Etwas mehr Setup, du brauchst einen GE-Asset. |

Für Story-Tags, die ins SaveGame gehören, **immer** GE-Variante. Für transiente (nur-aktuelles-Spiel) Tags reicht Loose.

{% hint style="warning" %}
**Bekannt**: AddTag greift aktuell nur auf den *lokalen* ASC; im Dedicated-Server-Setup musst du den Add-Call via Participant-RPC auf den Server-ASC routen. Siehe [Bridge & Lifecycle-Events](../runtime/bridge-events.md).
{% endhint %}

## Runtime-Trigger

Nichts Besonderes:

```cpp
Sub->StartDialogue(DA_Herbwoman_Visit, Player, Herbwoman);
```

Alles Weitere passiert innerhalb des Dialogs.

## Troubleshooting

### ApplyEffect zeigt keine Wirkung

* `EffectClass` ist leer – häufigster Fehler. Re-Check.
* Target hat keinen ASC – oder der ParticipantTag zeigt auf eine Participant-Komponente ohne zugeordneten Actor.
* Der GE ist Instant, aber dein Health-Attribut hat keine `PostGameplayEffectExecute`-Logik, die Current = Max klampt.

### Tag wird gesetzt, aber Branch nimmt trotzdem den Default-Pfad

* Du schreibst den Tag über Loose, und der Branch prüft am nächsten Frame (Scope-Stack-Pop). In seltenen Cases cached MayDialogue den Tag-State pro Frame – das ist noch ein Backlog-Item. Workaround: GE-Variante nutzen.

### CheckAttribute liefert nie Passed

* `Attribute` ist nicht korrekt angebunden. Im Attribute-Dropdown muss exakt dein `UVHSAttributeSet::Health`-FNames-Pair erscheinen. Wenn es leer ist, ist das AttributeSet nicht im ASC registriert.

## Nächster Schritt

* [Eigene GAS-Nodes erstellen](../gas/extending.md) – z.B. ein Requirement, das auf *Kombinationen* von Tags prüft.
* [Persistence → Participant-Memory](../persistence/participant-memory.md) – damit `PotionsBought` Spielstände überlebt.
