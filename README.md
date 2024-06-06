# PROMISE
Ein **framework** für die Anwendungsentwicklung, das die Entwicklung komplexer **language-based interactions** unter Verwendung von Konzepten der **state machine modelling** unterstützt.

Mit PROMISE können Sprachmodelle effektiver und effizienter genutzt und ihr Verhalten besser kontrolliert werden. Dies wird erreicht, indem eine modellgesteuerte, dynamische Prompt-Orchestrierung entlang hierarchisch verschachtelter Zustände ermöglicht wird, während Bedingungen und Aktionen, die mit bestimmten Interaktionssegmenten verbunden sind, einbezogen werden.

## Inhaltsübersicht
- [Why](#why)
- [What](#what)
- [How](#how-single-state-interaction)
- [Code](#code-single-state-interaction)
- [Getting Started](#getting-started)
- [Stepping Up](#stepping-up-multi-state-interactions)


## Why
Seit dem Aufkommen leistungsfähiger Sprachmodelle, das durch ihren jüngsten Durchbruch noch verstärkt wurde, sind die Erwartungen an immer komplexere Gesprächsinteraktionen zwischen Menschen und Maschinen rasch gestiegen. Dies unterstreicht die Notwendigkeit, die Durchführbarkeit und den Wert solcher Interaktionen untersuchen zu können. Während jedoch die Fähigkeiten von Sprachmodellen beeindruckende Fortschritte machen, hinkt die Fähigkeit, ihr Verhalten und ihre Konsistenz zu kontrollieren, hinterher.

LMs von Grund auf zu trainieren, um einen bestimmten Zweck zu erfüllen, ist ressourcenintensiv und für typische Entwicklungsprojekte oft unpraktisch. Die Feinabstimmung kann zwar die Reaktionen der LM anpassen, erfordert aber eine sorgfältige Datenvorbereitung, was ein schnelles, iteratives Experimentieren erschwert. Im Gegensatz dazu ermöglicht das Prompt-Engineering die Umgehung traditioneller Engpässe beim Vortraining und der Feinabstimmung. Die Spezifikation einer komplexen Interaktion bringt jedoch komplexe Prompts mit sich, denen es an Kontrollierbarkeit und Zuverlässigkeit mangelt. Letztlich wird keiner der beiden Ansätze den Herausforderungen gerecht, die entstehen, wenn komplexe Interaktionen entwickelt, in Informationssysteme integriert, in Varianten implementiert und iterativ verbessert werden sollen.

## What
Wir schlagen daher PROMISE (Prompt-Orchestrating Model-driven Interaction State Engineering) vor, einen Rahmen für die Anwendungsentwicklung, der den Bedarf an mehr Unterstützung für den schnellen Entwurf und die Implementierung komplexer dialogischer Interaktionen erfüllt. PROMISE überbrückt die Lücke zwischen den Anforderungen solcher Interaktionen und der Verwendung von Sprachmodellen, um sie zu ermöglichen. Die Unterstützung des Rahmens basiert auf einem Modell, das ein breites Spektrum von Anforderungen erfassen und die Verwendung von Sprachmodellen effektiv steuern kann, während gleichzeitig deren volle Fähigkeiten genutzt werden.

## How (Single-State Interaction)
Bei dem folgenden Gespräch handelt es sich um ein tägliches Check-in-Gespräch mit Patienten, die ein Gesundheitsinformationssystem nutzen. Ziel solcher Interaktionen ist es, das Wohlbefinden der Patienten in Bezug auf ihre chronische Erkrankung und ihren Therapieplan zu beurteilen.

<p align="center">
 <img alt="Check-in interaction with patients using a health information system" src=".readme/singlestateconversation-ui.png">
</p>

Bei PROMISE wird der folgende Zustandsautomat verwendet, um diese Interaktion zu entwerfen und umzusetzen.

<p align="center">
 <img alt="Check-in interaction with patients using a health information system" src=".readme/singlestatemodel.png">
</p>

Der **state** ist mit der **state prompt** „Als digitaler Therapiecoach, ...“ versehen, die zur Steuerung des LM verwendet wird, während sich die Interaktion in diesem Zustand befindet. Der ausgehende **transition** , der zum Endknotenpunkt führt, ist mit den Aufforderungen „Informationen bereitgestellt“, „Keine offenen Fragen“ und „Zusammenfassen“ versehen. Diese Aufforderungen steuern den LM bei der Auswertung des Gesprächs in Bezug auf **triggers**, **guards**, und **actions**. PROMISE stellt auf transparente Weise komplexere Prompts aus solchen einfachen Prompts zusammen, die an Zustände und Übergänge gebunden sind.

Diese einfache Beispielanwendung veranschaulicht ein Hauptmerkmal des erweiterten Zustandsmodells von PROMISE. Während der Zustands-Prompt verwendet wird, um die Generierung von Antworten an den Benutzer zu steuern, wenn sich die Interaktion in einem bestimmten Zustand befindet, werden separate Prompts verwendet, um die Entscheidung zu steuern, ob die Interaktion in einen anderen Zustand übergehen soll, sowie die Aktionen, die beim Übergang ausgeführt werden sollen. Die Verwendung von Prompts für solche Entscheidungen und Aktionen ermöglicht eine semantisch umfassende Steuerung von Interaktionsabläufen, z. B. auf der Grundlage von Gesprächsinhalten, die sich über mehrere Benutzeräußerungen hinweg manifestieren.

## Code (Single-State Interaction)
Eine Interaktion wie die durch das obige Zustandsmodell spezifizierte wird durch die Erstellung von Instanzen der Zustandsmodellkonzepte **State** und **Transition**. Ein **State** wird wie folgt erzeugt,

```
State state = new State(
    "As a digital therapy coach, check in with your patient...",
    "Check-In Interaction",
    "...compose a single, very short message to initiate...",
    List.of(transition)
);
```

wobei der als Teil der Liste angegebene **Transition** wie folgt erstellt wird.

```
Storage storage = new Storage();
Decision trigger = new StaticDecision(
    "Review the conversation...decide if...patient provided..."
);
Decision guard = new StaticDecision(
    "Review the conversation...decide if...no open issues..."
);
Action action = new StaticExtractionAction(
    "Summarize the conversation, highlighting...",
    storage,
    "summary"
);
Transition transition = new Transition(
    List.of(trigger, guard),
    List.of(action),
    new Final()
);
```

In der Regel umhüllt ein **Agent** den Zustandsautomaten und stellt die Funktionen bereit, die für die Integration der Interaktion mit einem Informationssystem erforderlich sind, z. B. mit einem REST-Controller, wenn PROMISE zur Bedienung einer Webanwendung verwendet wird.

```
Agent agent = new Agent(
    "Digital Companion",
    "Daily check-in conversation.",
    state
);
String conversationStarter = agent.start();
String response = agent.respond(
    "I am handling the fasting quite well."
);
```

## Getting Started

#### 1. Requirements (Anforderungen)
- Haben Sie ein JDK?
    - Testen Sie mit ***javac --version*** in Ihrer Konsole.
    - Wenn nicht, holen Sie es sich von https://dev.java/download/.
- Wurde die Umgebungsvariable JAVA_HOME gesetzt?
- Verfügen Sie über MySQL?
    - Holen Sie sich den ***MySQL Community Server***.
    - Denken Sie an **[PASSWORD]**, wenn Sie ihn nach der Installation konfigurieren.
    - Optional erhalten Sie ***MySQL Workbench*** , um direkt auf die Datenbank zuzugreifen.
- Verwenden Sie Visual Studio Code?
    - Holen Sie sich das ***Extension Pack for Java***.
    - Holen Sie sich das ***Spring Boot Extension Pack***.

#### 2. Set Up (Einrichten)
- Erstellen Sie eine Datenbank mit dem Namen **[DB_NAME]**.
- Im Projektordner ***src/main/ressources/***, ...
    - kopieren Sie beide Eigenschaftsvorlagen, benennen Sie sie um (entfernen Sie ***.template***).
    - ***application.properties***: Setzen Sie **[DB_NAME]** in der Verbindungsurl.
    - ***application.properties***: Setzen Sie **[PASSWORD]**.
    - ***openai.properties***: Wählen Sie openai vs. azure openai und legen Sie den API-Schlüssel fest.

***Definition of Done (erledigt)***:
Wenn Sie es erstellen können (z.B., Maven:statefulconversation:Plugins:spring-boot:run)

#### 3. Interaction
- Führen Sie einen vorhandenen Unit-Test in ***src/test/java/.../bots/*** aus (z. B. SingleStateInteraction)
- ODER erstellen Sie Ihren eigenen Unit-Test in ***src/test/java/.../.../***
    - Unit Test erstellt Agent und speichert ihn in der Datenbank
    - Führen Sie Ihren eigenen Unit-Test aus
- Starten Sie das Back-End (z.B., Maven:statefulconversation:Plugins:spring-boot:run)
- Finde **[UUID]** des Agenten: http://localhost:8080/all
- Interagieren Sie mit: http://localhost:8080/?[UUID]

## Stepping Up: Multi-State Interactions (Nach oben gehen: Mehrstaatliche Interaktionen)

Die folgende Assistenten-Patienten-Interaktion ist ein stark vereinfachtes, minimales Beispiel für die Notwendigkeit, mehrere Ziele in einer Gesprächsinteraktion zu erreichen. Die Interaktion wird ausgelöst, weil der Patient eine Therapieaktivität (Schwimmen) nicht abgeschlossen hat. Das erste Ziel dieser Interaktion besteht darin, den Grund für das Scheitern des Patienten zu erfahren (hellgrau), und das zweite Ziel besteht darin, Anpassungen an der Therapieaktivität vorzunehmen, um die Adhärenz des Patienten zu erhöhen (dunkelgrau).

<p align="center">
 <img alt="Check-in interaction with patients using a health information system" src=".readme/multistateconversation.png">
</p>

Der folgende Zustandsautomat modelliert diese Interaktion.

<p align="center">
 <img alt="Check-in interaction with patients using a health information system" src=".readme/multistatemodel.png">
</p>

Dieser Zustandsautomat enthält drei Neuerungen im Vergleich zu der oben vorgestellten Interaktion mit einem Zustand.
1. Mehrere Staaten folgen aufeinander, um einen **conversational flow** zu realisieren
2. **Special-purpose states** mit vordefiniertem Gesprächsverhalten sind beteiligt
3. Es gibt einen **outer state** der eine Folge von Innenzuständen enthält

#### Conversational Flows (Gesprächsfluss)
Die Fähigkeit, Zustandsfolgen zu erstellen, folgt aus der Fähigkeit, Übergänge zu erstellen. In diesem Beispiel beginnt die Interaktion mit einem Zustand, in dem der Patient nach den Gründen für das Versäumen der Schwimmaktivität gefragt wird. Wenn der Patient einen ausreichenden Grund angibt, geht die Interaktion in den nächsten Zustand über, in dem Optionen angeboten werden. Sobald sich der Patient für eine dieser Optionen entschieden hat, geht die Interaktion zum letzten Knoten weiter.

Im Allgemeinen kann jeder Zustand eine beliebige Anzahl von ausgehenden Übergängen haben. Jeder Übergang wird durch eigene Entscheidungen ausgelöst und überwacht, wird von eigenen Aktionen begleitet und bezieht sich auf einen beliebigen anderen Zustand. Folglich unterstützt PROMISE die Erstellung beliebiger gerichteter und möglicherweise zyklischer Graphen von Interaktionsflüssen.

#### Special-Purpose States
PROMISE enthält eine Bibliothek mit speziellen Zuständen, die wiederkehrende Anforderungen erfüllen. Ein Zustand zur Abfrage von Aktivitätslücken bewertet beispielsweise die Gründe des Patienten für das Versäumen einer Aktivität. Um ihn zu instanziieren, geben die Entwickler einfach die verpasste Aktivität an. In ähnlicher Weise akzeptiert ein Single-Choice-Status eine Liste von Wahlmöglichkeiten, die dem Patienten zur Verfügung gestellt werden und von denen er eine auswählen soll. Solche Zustände kapseln vordefinierte Aufforderungen und Übergänge, wodurch der Entwicklungsaufwand effektiv reduziert wird. Diese Bibliothek enthält auch Zustände, Auswahlmöglichkeiten und Aktionen, die eine erweiterte Generierung von Abfragen unterstützen, z. B. für die Abfrage von Dokumenten, Datenbanken oder anderen Wissensdatenbanken und die Zusammenführung von Abfrageergebnissen in Zustands- und Übergangsaufforderungen.

#### Outer States
PROMISE ist in der Lage, verschachtelte Gespräche zu unterstützen, indem Zustandsautomaten spezifiziert werden, die sich auf verschiedenen Ebenen - scheinbar gleichzeitig - verhalten können. In unserem Beispiel kann der Patient in jeder Phase der inneren Interaktion angeben, dass er die Interaktion beenden möchte. In PROMISE verfolgt ein äußerer Zustand das gesamte Gespräch in allen inneren Zuständen, da er seine eigenen Äußerungen für alle inneren Zustände beibehält. Daher reagiert jeder mit einem äußeren Zustand verbundene Übergang auf ein größeres Interaktionssegment, wenn Entscheidungen getroffen oder Aktionen durchgeführt werden. 

Wenn ein äußerer Zustand über einen eigenen Zustands-Prompt verfügt, wird dieser Prompt außerdem automatisch an die Zustands-Prompts aller seiner inneren Zustände angehängt. In diesem Beispiel ist die Rollenaufforderung „Als digitaler Therapiecoach, ...“ an den äußeren Zustand angehängt und muss daher nicht in allen inneren Zuständen wiederholt werden. Dies ermöglicht es Entwicklern, partielle Gesprächsverhaltensweisen zu spezifizieren, die umfangreichere Segmente betreffen, wie z. B. die Verwendung von Überredungsstrategien.

#### Interaction Storage
PROMISE bietet einen einfachen schlüsselwertbasierten Speicher, der einem Zustandsautomaten (Agenten) zugeordnet werden kann. Auf einen solchen Speicher kann jeder Zustand oder Übergang zugreifen, um Werte während der gesamten Interaktion zu speichern oder abzurufen. Zustände, Übergangsentscheidungen und -aktionen werden mit den Schlüsseln instanziiert, die sie zum Zeitpunkt der Interaktion verwenden, um Informationen abzurufen und in ihre Prompts einzuspeisen oder um Informationen zu speichern und sie für nachfolgende Zustände, Entscheidungen oder Aktionen verfügbar zu machen.

Diese Speicherung kann auch genutzt werden, um Informationen aus anderen Systemkomponenten in eine Interaktion einzubringen. Wenn zum Beispiel der Grund für das Scheitern eines Patienten bei einer Therapiemaßnahme ermittelt wurde und auf eine Wissensdatenbank zugegriffen wurde, um eine Anpassung der Maßnahme zu finden, würde diese Anpassung in den Speicher aufgenommen werden. Die Interaktion kann sie dann von dort abrufen und in ein Gespräch mit dem Patienten einbringen. Alternativ kann dieser Speicher auch verwendet werden, um Informationen aus dem Gespräch an andere Systemkomponenten weiterzugeben, z. B. eine Zusammenfassung einer täglichen Check-in-Interaktion, die an das Patienteninformationssystem des Arztes weitergeleitet wird.