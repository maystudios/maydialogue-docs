---
description: Wie PersistentMemory auf der Participant-Komponente funktioniert und wann du was nutzt.
---

# Participant-Memory

`PersistentMemory` auf der `UMayDialogueParticipant`-Komponente ist ein typisiertes Dictionary, das Dialog-übergreifend gespeichert bleibt. Es ist der richtige Ort für Dinge wie "Hat der Spieler diesen NPC schon getroffen?" oder "Wie oft wurde er angesprochen?".

## Fünf unterstützte Typen

| Typ | Beispiel-Variable | Setter | Getter |
| --- | --- | --- | --- |
| `bool` | `HasMet`, `IsAngry` | `SetPersistentBool` | `GetPersistentBool` |
| `int32` | `MeetingCount`, `InsultCount` | `SetPersistentInt` | `GetPersistentInt` |
| `float` | `Reputation`, `LastPrice` | `SetPersistentFloat` | `GetPersistentFloat` |
| `FString` | `LastTopicDiscussed` | `SetPersistentString` | `GetPersistentString` |
| `FGameplayTag` | `LastChoseTag` | `SetPersistentTag` | `GetPersistentTag` |

Alle Getter akzeptieren einen `DefaultValue`-Parameter: wenn die Variable noch nicht existiert, wird der Default zurückgegeben (kein Crash, kein Log-Warn).

{% hint style="info" %}
**Variable existiert nicht?** Beim Schreiben wird sie automatisch angelegt. Beim Lesen wird der Default zurückgegeben. Du musst nie vorher prüfen, ob eine Variable existiert.
{% endhint %}

## Im Dialog schreiben und lesen

Der einfachste Weg: `SetVariable`-Nodes und `GetVariable`-Nodes im Dialog direkt, mit Scope `Participant`. Dann passiert alles deklarativ im Dialog-Asset, ohne Blueprint-Glue.

> 📸 **Bild-Platzhalter:** `memory-set-in-dialogue.png` — SetVariable-SideEffect auf einer SayLine, Scope = Participant.
> *Setup:* SayLine "Schön, dich kennenzulernen!" im MayDialogue-Editor auswählen. SideEffects-Array aufgeklappt: `Set Variable`-Sub-Node sichtbar, `Variable Name = HasMet`, `Scope = Participant`, `Type = Bool`, `Value = true`. Details-Panel rechts zeigt dieselben Felder.

## Aus Blueprint-Code lesen und schreiben

Wenn dein Quest-System, dein Achievement-System oder ein anderer Actor den Memory-Zustand setzen soll:

```text
[Get Component by Class] → UMayDialogueParticipant
  │
  ├─ [Set Persistent Bool]   VarName="HasMet"        Value=true
  ├─ [Set Persistent Int]    VarName="MeetingCount"  Value=GetPersistentInt("MeetingCount")+1
  └─ [Get Persistent Bool]   VarName="HasMet"        Default=false  →  Rückgabewert
```

> 📸 **Bild-Platzhalter:** `memory-blueprint-getset.png` — Blueprint-Graph: Participant-Komponente holen, SetPersistentBool und GetPersistentInt aufrufen.
> *Setup:* BP-Graph im NPC-Blueprint. `Get Component by Class` → `Set Persistent Bool` (VarName="HasMet", Value=true). Zweite Kette: `Get Persistent Int` (VarName="MeetingCount", Default=0) → `Integer + 1` → `Set Persistent Int` (VarName="MeetingCount"). Alle Pins verbunden.

## Wann nutze ich was?

| Situation | Empfehlung |
| --- | --- |
| Erster Kontakt mit NPC merken | `SetPersistentBool("HasMet", true)` beim ersten Dialog-End |
| Anzahl Gespräche zählen | `SetPersistentInt("MeetingCount", GetPersistentInt+1)` |
| Letztes Gesprächsthema für Folge-Dialog | `SetPersistentString("LastTopic", …)` |
| Quest-Stage eines NPC-Gesprächspfads | `SetPersistentInt("QuestPhase", …)` |
| Komplexe structs, Inventar-Snapshots | Eigene `UPROPERTY(SaveGame)` in einer Participant-Blueprint-Subklasse |

{% hint style="warning" %}
**Typ-Mismatch:** Wenn du `GetPersistentBool` auf einer Variable aufrufst, die als Int gespeichert wurde, erhältst du den Default-Wert — keine Konvertierung. Halte Typ und Verwendungsstelle konsistent.
{% endhint %}

## Event: OnVariableChanged

Die Komponente sendet `OnVariableChanged` jedes Mal, wenn eine Persistent-Variable gesetzt wird:

```text
[Participant Component]
  OnVariableChanged (VarName, Scope, Type, NewValueAsString)
    │
    └─ Bind in deinem System-Blueprint
         → Prüfe VarName == "IsAngry" und NewValue == "true"
         → Trigger Combat / AI-State-Change
```

> 📸 **Bild-Platzhalter:** `memory-event-binding.png` — Blueprint: OnVariableChanged-Delegate binden und auf "IsAngry" reagieren.
> *Setup:* BP-Graph im NPC-Blueprint, `BeginPlay`-Event. `Get MayDialogue Participant Component` → `Bind Event to On Variable Changed` → Custom-Event `OnVarChanged` mit Parametern `VarName (Name), Scope, Type, NewValue (String)`. Im Event-Body: `Branch`: VarName == "IsAngry" AND NewValue == "true" → `Set AI State: Combat`.

## Lebenszyklus

| Zeitpunkt | Was passiert mit PersistentMemory |
| --- | --- |
| Actor-Spawn | PropertyBag ist leer (oder aus SaveGame geladen) |
| Während Dialog | Getter/Setter aktiv über Nodes oder Code |
| Actor-Destroy | Memory geht verloren, wenn vorher nicht gespeichert |
| Nach QuickSave | Memory in Slot gesichert |
| Nach QuickLoad | Memory aus Slot wiederhergestellt |

## Komplexere Daten

Die PropertyBag unterstützt nur die fünf Grundtypen. Für komplexere Strukturen (z.B. ein Dictionary von Item-IDs) legst du eine **Blueprint-Subklasse von `UMayDialogueParticipant`** an und erweiterst sie um eigene Variablen:

- **Im Blueprint:** Lege eigene Variablen in der Variable-Detail-Ansicht an und aktiviere für jede die `Save Game`-Checkbox. Diese Variablen werden automatisch vom UE-Standard-SaveGame-Archiv mitgespeichert.
- **In C++:** Füge eigene `UPROPERTY(SaveGame)`-Felder in der Subklasse hinzu — das Ergebnis ist dasselbe.

Setze diese Subklasse dann als Participant-Klasse deines Actors.
