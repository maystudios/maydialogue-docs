---
description: Das Dialog-UI-Theme zur Laufzeit umschalten — pro Level mit einem Drop-in-Actor, ereignisgesteuert mitten im Gespräch oder programmatisch aus Blueprint/C++.
---

# Theme-Wechsel zur Laufzeit

## Szenario

Dein Spiel liefert mehrere Dialog-Optiken aus — einen schmuddeligen Horror-Rahmen, eine weiche Visual-Novel-Pille, ein vergoldetes RPG-Panel — und du willst pro Level eine andere oder *im laufenden Spiel* zwischen ihnen umschalten. MayDialogue macht beides erstklassig:

* **`AMayDialogueThemeSetter`** — ein Drop-in-Level-Actor. Wähle eine Theme-Widget-Klasse, und jeder Dialog in diesem Level nutzt sie. Kein Code, kein Blueprint. Optional mappst du Event-Tags auf Themes für **Live-Wechsel mitten im Gespräch**.
* **`UMayDialogueSubsystem::SetDialogueWidgetClassOverride`** — ein welt-bezogenes Widget-Klassen-Override, das du aus Blueprint oder C++ für vollständig programmatisches Umschalten aufrufst (Einstellungsmenü, Optik pro NPC, geskriptete Cutscene).

Beide Wege setzen dasselbe welt-bezogene Override. Liegt bereits ein Dialog auf dem Bildschirm, **wechselt** das Setzen die UI sofort — das alte Widget wird abgebaut, das neue gespawnt und an das laufende Gespräch neu gebunden. Der **Theme-Kiosk** der Showcase-Map zeigt den ereignisgesteuerten Pfad von Anfang bis Ende: Der **Curator**-Dialog bietet drei Choices, jede feuert einen `FireEvent`-Seiteneffekt (`MayDialogue.Demo.Theme.Horror`, `MayDialogue.Demo.Theme.VN`, `MayDialogue.Demo.Theme.RPG`), und der ThemeSetter des Levels färbt die UI auf der Stelle neu ein.

## Was du lernst

* Einen `AMayDialogueThemeSetter` für ein **statisches Theme pro Level** (null Code) ins Level ziehen.
* Eine `EventThemes`-Tabelle an diesem Actor anlegen, sodass ein `FireEvent`-Tag das Theme **live mitten im Gespräch** umschaltet.
* `SetDialogueWidgetClassOverride` aus Blueprint/C++ für **programmatisches** Umschalten aufrufen (Menüs, pro NPC, Cutscenes).
* (Anhang) Das vollständig manuelle Muster — dein eigenes `UMayDialogueWidget` mit `BindToInstance` verwalten — wenn du volle Kontrolle brauchst.

## Voraussetzungen

* [Eigenes UMG-Widget anbinden](custom-umg-widget.md) abgeschlossen — du weißt, dass ein Theme eine `UMayDialogueWidget`-Subklasse ist, deren `BindWidgetOptional`-Slots auf themenbezogene Komponenten-Widgets verdrahtet sind.
* Eine oder mehrere Theme-Widget-Klassen bereit (deine eigenen Subklassen oder die mitgelieferten `WBP_MayDlg_Theme_Horror` / `_VN` / `_RPG`).

{% hint style="info" %}
**Präzedenz.** Das Welt-Override gewinnt immer. Der UI-Entscheidungsbaum des Subsystems lautet: **Welt-Override > `bUseSlateDialogueWidget` > `DefaultDialogueWidgetClass` > eingebauter `UMayDialogueWidget`-Fallback**. Ein ThemeSetter (oder ein `SetDialogueWidgetClassOverride`-Aufruf) überschreibt also *sowohl* das Slate-Debug-Widget *als auch* den projektweiten Default. Das Override ist **welt-bezogen** — es lebt und stirbt mit dem Subsystem des Levels, sodass das Reisen in ein Level ohne ThemeSetter automatisch auf den Projekt-Default zurückfällt.
{% endhint %}

## Weg A — `AMayDialogueThemeSetter` (empfohlen)

### A1. Statisches Theme pro Level (kein Code)

1. Im Level: **Place Actor → MayDialogue Theme Setter** (`AMayDialogueThemeSetter`).
2. Im Details-Panel **Theme Widget Class** auf das Theme für dieses Level setzen (z. B. `WBP_MayDlg_Theme_Horror`).

Das ist das ganze Setup. In `BeginPlay` ruft der Actor `SetDialogueWidgetClassOverride(ThemeWidgetClass)` auf, sodass jeder Dialog in diesem Level in der gewählten Optik öffnet — ohne die Project Settings anzufassen. Lass **Theme Widget Class** leer, um den Actor zum No-op zu machen (es wird die Projekt-Default-UI verwendet). Die vier ausgelieferten Sample-Maps tragen jeweils einen davon (Horror-Map → `_Horror`, VN-Map → `_VN`, RPG-Taverne + Showcase → `_RPG`).

> 📸 **Bild-Platzhalter:** `runtime-theme-switch-themesetter.png` — `AMayDialogueThemeSetter` im Level ausgewählt, Details-Panel zeigt `Theme Widget Class = WBP_MayDlg_Theme_Horror`.
> *Setup:* Horror-Sample-Map öffnen. Den MayDialogue-Theme-Setter-Actor im World Outliner auswählen; Screenshot des Details-Panels mit ausgefülltem `Theme Widget Class`-Feld.

### A2. Live-Wechsel, ereignisgesteuert (`EventThemes`)

Derselbe Actor kann das Theme *mitten im Gespräch* als Reaktion auf ein `FireEvent` umschalten. Fülle die **Event Themes**-Map (`TMap<GameplayTag, SoftClass<MayDialogueWidget>>`): Wenn *irgendein* Dialog in der Welt einen dieser Tags feuert, wird das gemappte Theme sofort angewendet.

Genau so funktioniert der Showcase-**Theme-Kiosk**:

| Event-Tag (`FireEvent`) | Theme-Widget-Klasse |
|---|---|
| `MayDialogue.Demo.Theme.Horror` | `WBP_MayDlg_Theme_Horror` |
| `MayDialogue.Demo.Theme.VN` | `WBP_MayDlg_Theme_VN` |
| `MayDialogue.Demo.Theme.RPG` | `WBP_MayDlg_Theme_RPG` |

Im Dialog gibst du dann jeder PlayerChoice einen **FireEvent**-Seiteneffekt (oder einen eigenständigen **FireEvent**-Action-Node) mit dem passenden Tag:

| Choice | FireEvent `EventTag` |
|---|---|
| „Zeig mir den Horror-Rahmen" | `MayDialogue.Demo.Theme.Horror` |
| „Zeig mir die Visual-Novel-Optik" | `MayDialogue.Demo.Theme.VN` |
| „Zeig mir das RPG-Panel" | `MayDialogue.Demo.Theme.RPG` |

Registriere die Tags in `DefaultGameplayTags.ini` (der Showcase-Content macht das bereits):

```ini
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.Horror")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.VN")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.RPG")
```

Wenn der Spieler eine Choice wählt, broadcastet `FireEvent` den Tag auf der laufenden Instanz; der ThemeSetter ist am Event-Delegate jedes Dialogs abonniert, findet den passenden Eintrag in `EventThemes` und ruft `SetDialogueWidgetClassOverride` — was die UI auf dem Bildschirm live zum neuen Theme wechselt und an das laufende Gespräch neu bindet. Kein Projekt-Glue, kein Blueprint.

> 📸 **Bild-Platzhalter:** `runtime-theme-switch-eventthemes.png` — `AMayDialogueThemeSetter`-Details-Panel mit aufgeklappter `Event Themes`-Map, drei Einträge mappen die `MayDialogue.Demo.Theme.*`-Tags auf die drei Theme-Widget-Klassen.
> *Setup:* Showcase-Map öffnen. Den MayDialogue-Theme-Setter des Theme-Kiosks auswählen; Screenshot des Details-Panels mit allen drei Einträgen in der `Event Themes`-Map.

> 📸 **Bild-Platzhalter:** `runtime-theme-switch-graph.png` — Curator-Dialog-Graph: eine PlayerChoice, deren drei Choices je einen FireEvent-Seiteneffekt tragen.
> *Setup:* Asset `DA_Showcase_Curator` geöffnet, PlayerChoice-Node ausgewählt, Details-Panel zeigt drei `UMayDialogueChoice`-Einträge; der erste aufgeklappt mit einem `FireEvent`-Seiteneffekt, dessen `EventTag = MayDialogue.Demo.Theme.Horror`.

## Weg B — `SetDialogueWidgetClassOverride` (programmatisch)

Wenn der Auslöser *kein* Dialog-Event ist — ein Einstellungsmenü-Button, eine bei der Interaktion gewählte Optik pro NPC, eine geskriptete Cutscene — rufst du das Subsystem direkt auf. Ein Aufruf erledigt alles: Er merkt sich das welt-bezogene Override und baut, falls ein Dialog läuft, die alte UI ab und bringt das neue Theme hoch, neu gebunden an das laufende Gespräch. Übergib **None**, um das Override zu löschen; der Projekt-Default kehrt beim nächsten Dialog-Start zurück.

**Blueprint:**

```text
[Get MayDialogue Subsystem]
   │
   ▼
[Set Dialogue Widget Class Override]
   └─ Widget Class: WBP_MayDlg_Theme_RPG   (Soft Class)
```

**C++:**

```cpp
void UThemeController::ApplyTheme(TSoftClassPtr<UMayDialogueWidget> ThemeClass)
{
    if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(this))
    {
        // Wechselt die UI auf dem Bildschirm live, wenn ein Dialog läuft;
        // sonst wird sie beim nächsten StartDialogue angewendet. None löscht sie.
        Sub->SetDialogueWidgetClassOverride(ThemeClass);
    }
}
```

Das Gegenstück `GetDialogueWidgetClassOverride()` liefert das aktuell aktive Override (oder None).

> 📸 **Bild-Platzhalter:** `runtime-theme-switch-setoverride-bp.png` — Einstellungsmenü-Blueprint: OnClicked eines Buttons → `Get MayDialogue Subsystem` → `Set Dialogue Widget Class Override` mit ausgewählter Soft-Theme-Klasse.
> *Setup:* Ein einfaches Options-Menü-Widget-Blueprint. `OnClicked (RpgThemeButton)` → `Get MayDialogue Subsystem` → `Set Dialogue Widget Class Override` (Widget Class = `WBP_MayDlg_Theme_RPG`).

### Variation / Weiter gehen

* **Themes pro NPC:** Abonniere das `OnAnyDialogueStarted` des Subsystems, inspiziere die Teilnehmer der neuen Instanz und rufe `SetDialogueWidgetClassOverride` mit dem Theme dieses NPCs auf, sodass jeder Charakter in seiner eigenen Optik öffnet.
* **Spieler-Menü-Theme-Picker:** Treibe `SetDialogueWidgetClassOverride` von einem Settings-Button — der Live-Wechsel macht die Änderung sichtbar, selbst wenn gerade ein Dialog offen ist.
* **Erst-Start-Default:** Das Override ist leer, bis du es setzt, also bestimmt der projektweite `DefaultDialogueWidgetClass` (oder das Slate-Debug-Widget) weiterhin den „noch kein Theme gewählt"-Start. Setze einen sinnvollen Default in den Project Settings und lass einen ThemeSetter oder `SetDialogueWidgetClassOverride` ab da übernehmen.

## Anhang — vollständig manuelles Muster (volle Kontrolle)

Wenn du die Widget-Lebensdauer selbst besitzen musst — z. B. die Dialog-UI in ein größeres HUD compositen oder eine von dir kontrollierte Animation über den Wechsel hinweg treiben — kannst du das Override ganz umgehen und dein eigenes `UMayDialogueWidget` verwalten. Binde es mit dem öffentlichen `BindToInstance(Instance)`-Aufruf an das laufende Gespräch und füge es selbst dem Viewport hinzu / entferne es:

```cpp
void UThemeController::SwapToManualWidget(TSubclassOf<UMayDialogueWidget> ThemeClass)
{
    UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(this);
    UMayDialogueInstance* Active = Sub ? Sub->GetActiveDialogue() : nullptr;

    // Das von uns verwaltete Widget abbauen (falls vorhanden).
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->UnbindFromInstance();
        CurrentThemeWidget->RemoveFromParent();
        CurrentThemeWidget = nullptr;
    }

    CurrentThemeWidget = CreateWidget<UMayDialogueWidget>(GetWorld(), ThemeClass);
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->AddToViewport();
        if (Active)
        {
            CurrentThemeWidget->BindToInstance(Active);
        }
    }
}
```

{% hint style="warning" %}
Wenn du vollständig manuell arbeitest, setze für denselben Dialog **nicht** zusätzlich ein Welt-Override — der Auto-Spawn des Subsystems und dein manuell verwaltetes Widget würden beide an dieselbe Instanz binden, und du sähest zwei Panels. Wähle einen Pfad: Lass das Override die UI besitzen (Weg A/B) oder besitze sie selbst (dieser Anhang) und lass das Override leer.
{% endhint %}

## Wie es zusammenpasst

```text
ThemeSetter (pro Level)          ──┐
ThemeSetter EventThemes (live)   ──┤
SetDialogueWidgetClassOverride   ──┴─▶  welt-bezogenes Override
   │                                        │
   │                              (Dialog läuft?)
   │                                 ja  → alte UI abbauen →
   │                                       neues Theme spawnen → BindToInstance
   │                                 nein → angewendet beim nächsten StartDialogue
   ▼
Präzedenz: Welt-Override > bUseSlateDialogueWidget
           > DefaultDialogueWidgetClass > eingebauter Fallback
```

## Troubleshooting

**Der ThemeSetter tut nichts.**
Seine `Theme Widget Class` ist nicht gesetzt (und `Event Themes` ist leer), also ist der Actor ein gewolltes No-op. Fülle mindestens eines aus. Prüfe außerdem, dass der Actor wirklich im geladenen Level liegt und sein `BeginPlay` lief.

**Ein `EventThemes`-Wechsel feuert nie.**
Der Dialog feuert den gemappten Tag nicht, oder der Tag ist falsch geschrieben. Prüfe, dass der `EventTag` des `FireEvent` der Choice exakt einem Schlüssel in der `Event Themes`-Map des ThemeSetters entspricht und dass der Tag in `DefaultGameplayTags.ini` registriert ist.

**Das Theme greift nur beim nächsten Gespräch, nicht beim aktuellen.**
Ein Live-Wechsel passiert nur, während tatsächlich ein Dialog läuft. Wenn du das Override setzt, bevor ein Dialog startet (oder nachdem er endete), wird es einfach beim nächsten `StartDialogue` angewendet — das ist erwartet.

**Zwei Dialog-Panels erscheinen gleichzeitig.**
Du hast das Welt-Override mit dem manuellen Anhang-Muster kombiniert, sodass sowohl ein auto-gespawntes Widget als auch dein eigenes Widget an dieselbe Instanz gebunden sind. Verwende nur einen Pfad (siehe Anhang-Warnung).
