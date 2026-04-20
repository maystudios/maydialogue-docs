# Verzweigungen mit Bedingungen

Dieses Rezept zeigt dir, wie du mit dem **Branch-Node** und einem **GAS-Requirement** einen Dialog-Pfad abhängig vom Spieler-Zustand aufspaltest. Das Muster ist der Standard-Weg, um Weltreaktivität in MayDialogue umzusetzen.

## Szenario

Ein Wachmann am Stadttor reagiert anders, je nachdem, ob der Spieler bereits den Stadt-Pass-Tag `Story.Pass.CityGate` besitzt:

* **Mit Pass** → „Willkommen zurück." → Tor öffnet.
* **Ohne Pass** → „Halt! Wer geht da?" → PlayerChoice mit Optionen zum Erwerb.

## Beteiligte Nodes

* 1 × [Entry](../nodes/core/entry.md)
* 1 × [Branch](../nodes/core/branch.md) mit 2 Branch-Points
* 2 × [SayLine](../nodes/core/say-line.md)
* 1 × [PlayerChoice](../nodes/core/player-choice.md)
* 1 × [Exit](../nodes/core/exit.md)

Dazu ein **Requirement**-Sub-Node vom Typ [HasTag](../gas/requirements.md#umaydlgrequirement_hastag) am ersten Branch-Point.

## Graph-Mock-Up

```
                         [Entry]
                            │
                            ▼
                   ┌─────────────────┐
                   │     Branch      │
                   │ BP1: HasTag(Pass)  ──────► [SayLine: "Willkommen zurück."] ─► [Exit]
                   │ BP2: <fallback>    ──────► [SayLine: "Halt! Wer geht da?"] ─► [PlayerChoice]
                   └─────────────────┘
```

## Schritt-für-Schritt

1. **Asset anlegen**: `DA_Guard_Gate`. Speaker: `Dialogue.Speaker.Guard`.
2. **Branch-Node einfügen**: Vom Entry-Output ziehen → *Create Node → Branch*.
3. **Erster Branch-Point** (`BranchPoints[0]`):
   * *Description*: *„Mit Pass"* (nur Editor-Label).
   * *Requirements* → *Add* → [HasTag](../gas/requirements.md#umaydlgrequirement_hastag)
     * `RequiredTag`: `Story.Pass.CityGate`
     * `bCheckOnInstigator`: `true`
4. **Zweiter Branch-Point** (`BranchPoints[1]`):
   * *Description*: *„Fallback"* – keine Requirements. Er wird der Default-Pfad.
5. **Erfolgs-Pfad** (Output von BP1):
   * SayLine *„Willkommen zurück. Tor ist offen."*
   * Direkt in einen Exit-Node (Completed).
6. **Ablehnungs-Pfad** (Output von BP2):
   * SayLine *„Halt! Wer geht da?"*
   * PlayerChoice mit zwei Choices (siehe unten).
7. **PlayerChoice-Node** konfigurieren:
   * Choice 1: *„Ich habe keinen Pass."* → Exit (Failed).
   * Choice 2: *„Ich suche den Torvorsteher."* → SayLine *„Dann geh zur Kammer links."* → Exit.
8. **Compile** und speichern.

## Branch-Auswertungslogik

Branch-Points werden **in Reihenfolge** ausgewertet. Der erste, dessen Requirements *Passed* sind, gewinnt.

| BranchPoint-Ergebnis | Verhalten |
| --- | --- |
| `Passed` | Nimm diesen Pfad, Branch ist abgeschlossen. |
| `FailedButVisible` | Überspringen, weiter zum nächsten Point. |
| `FailedAndHidden` | Überspringen, weiter zum nächsten Point. |

Der letzte BranchPoint ohne Requirements wirkt automatisch als *Default* / *Fallback*. Ohne Fallback und ohne Erfolgs-Match bricht der Branch mit `AbortDialogue` ab.

## GAS-Setup für den Tag

Damit das Requirement greift, muss der Spieler den Tag tatsächlich tragen. Typischer Setup:

```cpp
// Irgendwo, wenn der Quest-Schritt abgeschlossen wird
UAbilitySystemComponent* ASC = UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(Player);
ASC->AddLooseGameplayTag(FGameplayTag::RequestGameplayTag(TEXT("Story.Pass.CityGate")));
```

Oder über ein GameplayEffect, das den Tag in `GrantedTags` setzt – die Persistenz regelt dann dein Quest-System.

{% hint style="info" %}
Für **persistente** Tags (die ein Save überleben sollen) nutze lieber einen [Infinite-Duration GameplayEffect](../gas/actions.md) mit `GrantedTags`. `AddLooseGameplayTag` wird nicht automatisch persistiert.
{% endhint %}

## Runtime-Trigger

```cpp
void AGuardNPC::StartConversation(AActor* Player)
{
    UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld());
    if (Sub && Sub->CanStartDialogue(GuardDialogueAsset, Player, this))
    {
        Sub->StartDialogue(GuardDialogueAsset, Player, this);
    }
}
```

## Variante: Branch statt Requirement auf Choice

Manchmal willst du den **kompletten Pfad** abhängig machen (so wie hier am Gateway). Manchmal willst du nur eine **einzelne Choice** verstecken oder greyen. Beides ist möglich:

| Pattern | Nutze wann |
| --- | --- |
| `Branch + Requirement pro BranchPoint` | Ganze Gespräche oder Abschnitte davon. |
| `PlayerChoice + Requirement pro Choice` | Nur eine einzelne Entscheidung filtern. |

Details siehe [Requirement-Sub-Node](../nodes/sub-nodes/requirement.md).

## Troubleshooting

### Der Dialog nimmt immer den Fallback-Pfad

* Prüfe im [Debugger](../editor/debugger.md), ob der Tag tatsächlich am richtigen ASC hängt. Das *Instigator*-vs-*Target*-Flag ist der häufigste Fehler.
* Der Tag muss **exakt** matchen. `Story.Pass.CityGate` ≠ `Story.Pass.City.Gate`.

### Der Erfolgs-Pfad wird nie genommen, obwohl der Tag da ist

* `bCheckOnInstigator` steht auf `false` – Requirement prüft den NPC statt den Spieler.
* Der Actor hat keinen ASC und auch kein `IGameplayTagAssetInterface`. Requirement liefert dann immer Fail.

### Branch bricht mit „No BranchPoint passed" ab

* Letzter BranchPoint hat Requirements. Füge entweder einen leeren Default-Point hinzu oder setze auf dem Branch-Node `FailBehavior = Skip`.

## Nächster Schritt

* [GAS-getriebener Dialog](gas-driven-dialogue.md) – Tags nicht nur prüfen, sondern im Gespräch setzen.
* [Wiederverwendbare Dialog-Fragmente](linking-dialogues.md) – den Ablehnungs-Pfad als Link auslagern, damit mehrere Wachen denselben Pfad nutzen.
