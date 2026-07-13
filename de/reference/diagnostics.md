---
description: "Stabile MayDialogue-Validator-Regel-IDs, Bedeutung jedes Funds und passende Behebung."
---

# Validator-Diagnosen

Compiler und MCP-Authoring-Tools melden stabile `MDV.*`-Regel-IDs. Fehler blockieren die Kompilierung; Warnungen und Hinweise markieren gültiges, aber wahrscheinlich unbeabsichtigtes Verhalten. Behebe den Graph und kompiliere erneut, statt eine Regel zu unterdrücken.

## MDV.NullGraph

Der Validator hat kein Graph-Objekt erhalten. Öffne oder erstelle das Dialog-Asset neu und stelle sicher, dass eigener Editor-Code einen gültigen `UMayDialogueGraph` zur Validierung übergibt.

## MDV.EmptyGraph

Der Graph enthält keine Nodes. Füge Entry und Exit hinzu und verbinde den spielbaren Ablauf dazwischen.

## MDV.MissingEntry

Dem Graph fehlt ein Entry-Node. Jedes Dialog-Asset hat genau einen Einstiegspunkt; füge den Entry hinzu oder stelle ihn wieder her.

## MDV.MissingExit

Dem Graph fehlt ein Exit-Node, sodass eine Instanz möglicherweise nie endet. Füge einen Exit hinzu und verbinde alle vorgesehenen Endpfade mit ihm.

## MDV.MultipleEntries

Der Graph enthält mehr als einen Entry. MayDialogue unterstützt bewusst genau einen Einstiegspunkt pro Asset; behalte einen Entry und verwende Branch-, Link- oder SubGraph-Nodes für alternative Abläufe.

## MDV.UnconnectedPins

Ein erforderlicher Output-Pin ist nicht verbunden. Verbinde ihn mit dem nächsten Node oder mit Exit. Bei Player Choice muss jede wählbare Antwort verdrahtet sein.

## MDV.EmptyText

Einer Say Line oder Player Choice fehlt nutzbarer Inhalt, eine Choice ist leer oder unverbunden oder der Timeout-Index ist ungültig. Ergänze lokalisierbaren Text oder – wo unterstützt – Voice-Audio, mindestens eine gültige Choice, jede Verbindung und einen Index innerhalb des Choices-Arrays.

## MDV.UnreachableNodes

Ein Node ist von Entry aus nicht erreichbar. Verbinde ihn mit dem vorgesehenen Ablauf oder lösche ihn, wenn er veraltet ist; unerreichbare Nodes werden nie ausgeführt.

## MDV.MissingSpeakers

Einem sprecherabhängigen Node fehlt Speaker-/Target-Tag oder sein Tag steht nicht in der Speakers-Liste des Assets. Wähle einen deklarierten Tag und ergänze den Sprecher im Asset.

## MDV.SelfLoop

Ein Node-Output ist direkt mit demselben Node verbunden. Führe die Schleife über einen expliziten zustandsändernden Pfad oder entferne die versehentliche Verbindung.

## MDV.Deadlock

Entry oder ein anderer erreichbarer Node besitzt keinen Pfad zu einem Exit. Verdrahte einen Abschluss und stelle sicher, dass Zyklen eine tatsächlich erreichbare Ausstiegsbedingung haben.

## MDV.CyclicSubgraph

Ein Zyklus im aktuellen Graph hat keinen erreichbaren Exit. Ergänze einen Ausstiegszweig oder strukturiere die Schleife so um, dass der Dialog enden kann.

## MDV.CrossGraphCycle

SubGraph-Referenzen bilden über Graph-Grenzen hinweg einen Zyklus. Entferne eine rekursive Referenz oder ersetze sie durch eine explizite Schleife in einem einzelnen Graph mit erreichbarem Exit.

## MDV.Subgraph

Ein SubGraph-Node hat keinen zugewiesenen Graph, keine gültige Entry-Identität, referenziert sich selbst oder sein Ziel besitzt keinen Entry beziehungsweise Exit. Weise einen gültigen Subgraph zu und repariere dessen Grenz-Nodes.

## MDV.InvalidAssetReference

Ein Link-Node hat kein Zieldialog-Asset oder erzeugt eine nicht unterstützte Selbstreferenz. Weise ein anderes gültiges Dialog-Asset zu; bilde Schleifen als lokalen Graph-Ablauf statt per Selbst-Link ab.

## MDV.MissingRequirements

Ein Branch hat keine Bedingung und nimmt daher immer den True-Pfad. Füge mindestens ein Requirement hinzu oder ersetze den Branch durch eine direkte Verbindung.

## MDV.VariableTypeMismatch

Derselbe Variablenname wird mit unterschiedlichen MayDialogue-Typen geschrieben. Entscheide dich für Bool, Int, Float, String oder Tag und verwende Typ und Scope in allen Set-Variable-Stellen konsistent.

## MDV.AsyncWithoutContinuation

Ein asynchroner Node wie Wait, Play Animation oder Camera Focus hat keinen Output. Verbinde die Fortsetzung, damit die Instanz nach Abschluss der Operation weiß, wo sie weiterläuft.

## MDV.BranchPins

Ein erforderlicher True-, False- oder Default-Output ist nicht verbunden. Verdrahte jeden möglichen Ausgang oder passe die Konfiguration so an, dass der unbenutzte Pfad nicht gewählt werden kann.

## MDV.WaitNoOp

Ein Wait-Node besitzt weder eine sinnvolle Dauer noch ein konfiguriertes Event-Warten. Setze eine positive Dauer oder eine unterstützte externe Fortsetzung; andernfalls entferne den Node.

## MDV.RandomWeights

Eine Random Line hat keine Outputs oder ihre Gewichte passen nicht zur Output-Anzahl. Verdrahte mindestens einen Ausgang und gib exakt ein gültiges Gewicht pro Output an.

## MDV.CultureKey

Ein `VoicePerCulture`-Schlüssel ist kein erkannter Kulturname. Verwende eine Unreal-/IETF-Kultur wie `en`, `en-US`, `de` oder `de-DE` und setze Sprachfamilien-Fallbacks bewusst ein.

## MDV.CameraFocus

Mehrere Camera-Focus-Nodes beanspruchen inkompatibel die Steuerung des Sichtfelds. Lass nur den vorgesehenen Node den FOV auf diesem Pfad ändern oder deaktiviere den kollidierenden Override.

## MDV.ActionFields

Einem Action-Node fehlt ein erforderliches Asset, eine Klasse, ein Gameplay Tag, ein Variablenname oder Participant-Tag. Fülle das in der Diagnose genannte Feld; räumliche Aktionen benötigen außerdem ein gültiges Sprecher-/Participant-Ziel.

## MDV.SubNodeFields

Einem Requirement oder SideEffect fehlt eine erforderliche Variable, ein Tag, eine Klasse, ein Asset oder Participant-Ziel. Öffne das Sub-Node-Array des besitzenden Nodes und vervollständige das genannte Feld.

## MDV.UnlocalizedText

Sichtbarer Text wurde kulturinvariant angelegt und kann nicht lokalisiert werden. Gib ihn erneut als lokalisierbares `FText` ein und exportiere danach die Lokalisierungs-CSV erneut.
