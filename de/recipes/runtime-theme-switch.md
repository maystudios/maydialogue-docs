---
description: Das Dialog-UMG-Theme zur Laufzeit umschalten — ausgelöst durch ein FireEvent-Seiteneffekt aus dem Dialog selbst.
---

# Theme-Wechsel zur Laufzeit

## Szenario

Dein Spiel liefert mehrere Dialog-Optiken aus — einen schmuddeligen Horror-Rahmen, eine weiche Visual-Novel-Pille, ein vergoldetes RPG-Panel — und du willst im laufenden Spiel zwischen ihnen umschalten. Der „Theme-Kiosk" der Showcase-Map zeigt genau das: Der **Curator**-Dialog bietet drei Choices, jede feuert einen `FireEvent`-Seiteneffekt (`MayDialogue.Demo.Theme.Horror`, `MayDialogue.Demo.Theme.VN`, `MayDialogue.Demo.Theme.RPG`), und ein kleines Stück Projekt-Glue reagiert, indem es die Dialog-UI neu einfärbt.

Dieses Rezept dokumentiert den **funktionierenden Ansatz mit der ausgelieferten API**. Lies zuerst die Ehrlichkeits-Box unten: MayDialogue liefert **keinen** einzelnen „tausche das laufende Widget"-Aufruf, deshalb lautet das Muster „Theme-Klasse wählen, dann Widget (neu) erstellen" — und genau das macht der Kiosk wirklich.

{% hint style="warning" %}
**Was die ausgelieferte API kann und was nicht (gegen den Quellcode verifiziert, 1.0):**

* Es gibt **eine** Widget-Klassen-Einstellung — `UMayDialogueSettings::DefaultDialogueWidgetClass` (`TSoftClassPtr<UMayDialogueWidget>`). Ein „Theme" ist eine `UMayDialogueWidget`-Subklasse (z. B. `WBP_MayDlg_Theme_Horror`), deren `BindWidgetOptional`-Slots auf themenbezogene Komponenten-Widgets vorverdrahtet sind. Das Plugin auf ein Theme zeigen lassen = diese eine Klasse setzen.
* Das Subsystem spawnt dieses Widget **einmal**, beim ersten Dialog-Start, nach `AutoSpawnedUMGWidget`. Dieses Feld ist **privat ohne öffentlichen Getter oder Setter**, und es gibt **keine** `SetTheme()`- / `ApplyTheme()`- / `RebuildComponents()`-Methode auf `UMayDialogueWidget`. Du kannst das *bereits gespawnte* Auto-Widget also nicht über die öffentliche API an Ort und Stelle umstylen.
* Was du **mit ausgelieferter API kannst**: dein eigenes `UMayDialogueWidget` verwalten, es mit dem öffentlichen `BindToInstance(Instance)`-Aufruf binden und im Viewport hinzufügen/entfernen. Das ist der saubere Laufzeit-Wechsel. Der ehrliche Workaround für das *aktuelle* Gespräch ist „themenbezogenes Widget neu erstellen und neu binden"; für das *nächste* Gespräch ändert das Schreiben von `DefaultDialogueWidgetClass`, was der Auto-Spawn verwendet.
{% endhint %}

## Was du lernst

* Die drei Theme-Choices als `FireEvent`-Seiteneffekte an einer PlayerChoice anlegen.
* **Einmal** das aggregierte `OnAnyDialogueStarted` des Subsystems abonnieren, um jede neue `UMayDialogueInstance` einzufangen.
* Auf das pro-Instanz `OnDialogueEventFired` (ein `FGameplayTag`) reagieren und die Theme-Tags auf Widget-Klassen mappen.
* Dein eigenes themenbezogenes `UMayDialogueWidget` steuern, sodass der Wechsel sofort sichtbar ist — ganz ohne Plugin-seitige Widget-Swap-API.

## Voraussetzungen

* [Eigenes UMG-Widget anbinden](custom-umg-widget.md) abgeschlossen — du weißt, dass ein Theme eine `UMayDialogueWidget`-Subklasse ist.
* Die drei mitgelieferten Themes existieren unter `Content/Themes/{Horror,VN,RPG}/Widgets/WBP_MayDlg_Theme_*` (oder deine eigenen Subklassen).
* `Project Settings → MayDialogue → UI → Use Slate Dialogue Widget` ist **aus** (`bUseSlateDialogueWidget = false`), damit der UMG-Pfad aktiv ist.

## Schritt-für-Schritt

### 1. Theme-Event-Tags definieren

In `DefaultGameplayTags.ini` (der Showcase-Content registriert diese bereits):

```ini
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.Horror")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.VN")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.RPG")
```

### 2. Das Theme-Event aus dem Dialog feuern

Öffne den Curator-Dialog. Gib jeder Choice am PlayerChoice-Node einen **FireEvent**-Seiteneffekt (oder verdrahte einen eigenständigen **FireEvent**-Action-Node hinter die Choice). Das einzig relevante Feld ist der Tag:

| Choice | FireEvent `EventTag` |
|--------|----------------------|
| „Zeig mir den Horror-Rahmen" | `MayDialogue.Demo.Theme.Horror` |
| „Zeig mir die Visual-Novel-Optik" | `MayDialogue.Demo.Theme.VN` |
| „Zeig mir das RPG-Panel" | `MayDialogue.Demo.Theme.RPG` |

`FireEvent` broadcastet den Tag auf der laufenden Instanz — er erreicht jeden Listener, der am `OnDialogueEventFired` dieser Instanz hängt. Nichts im Dialog selbst berührt die UI; das Projekt-Glue (nächster Schritt) macht das Neu-Einfärben.

> 📸 **Bild-Platzhalter:** `runtime-theme-switch-graph.png` — Curator-Dialog-Graph mit einer PlayerChoice, deren drei Choices je einen FireEvent-Seiteneffekt tragen.
> *Setup:* Asset `DA_Showcase_Curator` geöffnet. PlayerChoice-Node ausgewählt, Details-Panel zeigt drei `UMayDialogueChoice`-Einträge; der erste aufgeklappt mit einem `FireEvent`-Seiteneffekt, dessen `EventTag = MayDialogue.Demo.Theme.Horror`.

### 3. Die Tags auf Theme-Widget-Klassen mappen

In deinem Theme-Controller (ein GameInstance-Subsystem, das Level-Blueprint oder ein kleiner Actor) baust du ein Lookup von Event-Tag → `UMayDialogueWidget`-Subklasse:

```text
MayDialogue.Demo.Theme.Horror → WBP_MayDlg_Theme_Horror
MayDialogue.Demo.Theme.VN     → WBP_MayDlg_Theme_VN
MayDialogue.Demo.Theme.RPG    → WBP_MayDlg_Theme_RPG
```

Im Blueprint ist eine `Map<GameplayTag, SoftClass<MayDialogueWidget>>`-Variable die einfachste Form. In C++ eine `TMap<FGameplayTag, TSubclassOf<UMayDialogueWidget>>`.

### 4. Jeden Dialog-Start abonnieren

Binde einmalig (z. B. in `BeginPlay`) an das aggregierte `OnAnyDialogueStarted` des Subsystems. Es übergibt dir die frisch erzeugte `UMayDialogueInstance` — dort liegt das Event-Delegate.

**Blueprint:**

```text
[Event BeginPlay]
   │
   ▼
[Get May Dialogue Subsystem]
   │
   ▼
[Bind Event to On Any Dialogue Started]
        └─ Event:  HandleDialogueStarted(Instance)
```

**C++:**

```cpp
void UThemeController::Init()
{
    if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
    {
        Sub->OnAnyDialogueStarted.AddDynamic(this, &UThemeController::HandleDialogueStarted);
    }
}
```

### 5. Bei jedem Start das pro-Instanz-Event binden

`OnAnyDialogueStarted` liefert die `UMayDialogueInstance`. Abonniere das `OnDialogueEventFired` dieser Instanz (ein `FGameplayTag`-Parameter):

**Blueprint:**

```text
[HandleDialogueStarted (Instance)]
   │
   ▼
[Instance → Bind Event to On Dialogue Event Fired]
        └─ Event: HandleThemeEvent(EventTag)
```

**C++:**

```cpp
void UThemeController::HandleDialogueStarted(UMayDialogueInstance* Instance)
{
    if (Instance)
    {
        Instance->OnDialogueEventFired.AddDynamic(this, &UThemeController::HandleThemeEvent);
    }
}
```

### 6. Theme anwenden — Widget neu erstellen und neu binden

Wenn der Theme-Tag eintrifft, schlägst du die Widget-Klasse nach und bringst das themenbezogene Widget hoch. Da es keine In-Place-Restyle-API gibt, entfernst du das vorherige Dialog-Widget, erstellst das themenbezogene und bindest es mit dem öffentlichen `BindToInstance`-Aufruf an die laufende Instanz:

**C++:**

```cpp
void UThemeController::HandleThemeEvent(const FGameplayTag& EventTag)
{
    const TSubclassOf<UMayDialogueWidget>* Found = ThemeClasses.Find(EventTag);
    if (!Found || !*Found)
    {
        return; // kein Theme-Tag — ignorieren
    }

    UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld());
    UMayDialogueInstance* Active = Sub ? Sub->GetActiveDialogue() : nullptr;

    // Das von uns verwaltete Widget abbauen (falls vorhanden).
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->UnbindFromInstance();
        CurrentThemeWidget->RemoveFromParent();
        CurrentThemeWidget = nullptr;
    }

    // Themenbezogenes Widget hochbringen und an das laufende Gespräch binden.
    CurrentThemeWidget = CreateWidget<UMayDialogueWidget>(GetWorld(), *Found);
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->AddToViewport();
        if (Active)
        {
            CurrentThemeWidget->BindToInstance(Active);
        }
    }

    // Die Wahl persistieren, damit auch der NÄCHSTE Dialog in diesem Theme auto-spawnt.
    if (UMayDialogueSettings* Settings = GetMutableDefault<UMayDialogueSettings>())
    {
        Settings->DefaultDialogueWidgetClass = *Found;
    }
}
```

Das Blueprint-Äquivalent: `Create Widget` (Class = die nachgeschlagene Theme-Klasse), `Add to Viewport`, dann `Bind To Instance` (die aktive Instanz aus `Get Active Dialogue`), und das zuvor erstellte Widget entfernen. Optional `DefaultDialogueWidgetClass` über einen kleinen C++-Helfer schreiben, damit das nächste Gespräch bereits themenbezogen startet.

> 📸 **Bild-Platzhalter:** `runtime-theme-switch-controller-bp.png` — Theme-Controller-Blueprint: OnAnyDialogueStarted → OnDialogueEventFired binden → Switch on Tag → Create Widget + Bind To Instance.
> *Setup:* GameInstanceSubsystem-Blueprint-Graph. `OnAnyDialogueStarted` rotes Event-Node → `Bind Event to On Dialogue Event Fired`. Darunter ein `Switch on GameplayTag` mit drei Pins (Horror/VN/RPG), jeder speist `Create Widget` → `Add to Viewport` → `Bind To Instance`.

### 7. Den Auto-Spawn den „noch kein Theme gewählt"-Start übernehmen lassen

Für den allerersten Dialog setzt du `DefaultDialogueWidgetClass` in den Project Settings auf eine sinnvolle Voreinstellung (z. B. `WBP_MayDlg_Theme_VN`). Das Subsystem spawnt sie beim ersten Dialog-Start automatisch; dein Controller übernimmt erst, sobald ein Theme-Event feuert. Wenn dein Controller das Widget von Anfang an besitzen soll, lass den pro-Dialog-Auto-Spawn stehen und ersetze ihn einfach in Schritt 6 — das Auto-Widget und dein Widget sind getrennte `UMayDialogueWidget`-Instanzen, also entbinde/entferne deins und lass das Auto-Widget in Ruhe (oder umgekehrt); vermeide nur zwei Widgets, die gleichzeitig an dieselbe Instanz gebunden sind.

## Wie es zusammenpasst

```text
Dialog-Choice → FireEvent(MayDialogue.Demo.Theme.X)
        │ (broadcast auf der laufenden Instanz)
        ▼
Instance.OnDialogueEventFired(EventTag)
        │ (dein Controller hört zu, gebunden via OnAnyDialogueStarted)
        ▼
Theme-Klasse nachschlagen → CreateWidget → AddToViewport → BindToInstance
        │
        └─ (optional) DefaultDialogueWidgetClass für das nächste Gespräch schreiben
```

## Variation / Weiter gehen

* **Reiner Settings-Wechsel (zwischen Gesprächen):** Wenn der *aktuelle* Dialog nicht mitten in der Zeile neu eingefärbt werden muss, überspringe Schritte 4–6 und schreibe einfach `DefaultDialogueWidgetClass`, wenn der Theme-Tag feuert. Das neue Theme erscheint beim nächsten `StartDialogue`. Einfacher, aber nicht live.
* **Spieler-Menü-Theme-Picker:** Treibe dieselbe `HandleThemeEvent`-Logik von einem Settings-Menü-Button statt einer Dialog-Choice — der Wechsel-Pfad ist identisch, sobald du einen Tag (oder eine Klasse) in der Hand hast.
* **Themes pro NPC:** Mappe einen Speaker-Tag (nicht einen Event-Tag) auf ein Theme und wähle die Widget-Klasse in `OnAnyDialogueStarted` anhand der Teilnehmer der Instanz, sodass jeder NPC in seiner eigenen Optik öffnet.

## Troubleshooting

**Nichts passiert, wenn ich eine Choice wähle.**
Die Choice hat keinen `FireEvent`-Seiteneffekt, oder der Tag ist falsch geschrieben. Prüfe, dass der `EventTag` an der Choice exakt `MayDialogue.Demo.Theme.*` entspricht und dass der Tag in `DefaultGameplayTags.ini` registriert ist.

**Das Event feuert, aber das Widget ändert sich nie.**
Wahrscheinlich versuchst du, das automatisch gespawnte Widget des Plugins umzustylen — das ist nicht exponiert. Stelle sicher, dass Schritt 6 *dein eigenes* `UMayDialogueWidget` erstellt und `BindToInstance` aufruft; das Auto-Widget ist privat und kann nicht an Ort und Stelle getauscht werden.

**Zwei Dialog-Panels erscheinen gleichzeitig.**
Du hast ein neues Widget hinzugefügt, ohne das alte zu entfernen (auto-gespawnt oder dein vorheriges Theme-Widget). Rufe `RemoveFromParent` + `UnbindFromInstance` auf dem zu ersetzenden Widget auf, bevor du das neue hinzufügst.

**Das Theme greift nur beim nächsten Gespräch, nicht beim aktuellen.**
Das heißt, du hast nur `DefaultDialogueWidgetClass` geschrieben (die reine Settings-Variante). Um das laufende Gespräch neu einzufärben, musst du das neue Widget wie in Schritt 6 erstellen und `BindToInstance` aufrufen.
