## How   to   - Zugr i ff auf Bas i s Images

## 1 .  Z i el

Diese Anleitung zeigt, wie Sie auf die Basis-Images der Plattform zugreifen können. Sie lernen, wie Sie diese Images in BuildProzessen (z.B. über ein Dockerfile) verwenden und wie Sie einen Pod mit diesen Images starten können.

## 2.  Voraussetzungen

- Zugang zur RedHat OpenShift Container Platform
- Grundlegende Kenntnisse über Container und Kubernetes/OpenShift
- Berechtigungen zum Erstellen und Ausführen von Pods in Ihrem Namespace

## 3.  Verf ü gbare   Bas i s-Images

Alle folgenden Images sind im Cluster frei und ohne Einschränkungen verfügbar. Sie können für das Erstellen von Containern genutzt werden.

Red Hat stellt Universal Base Images bereit, die kostenlos und lizenzfrei verwendbar sind.

- Infos zu UBIs
- UBIs für Docker Nutzer:

Viele der offiziellen und subskriptionsfreien RedHat Universal Build Images (UBI) sind auf der Plattform verfügbar. Diese Images sind in der Quay-Organisation `plattform-base-images` unter folgender URL erhältlich:

```
1 cpk.quay.asg.net/plattform-base-images/<repo>
```

## Core   Base   Images

## UBI  9   (Latest) :

- cpk.quay.asg.net/plattform-base-images/ubi9/ubi
- cpk.quay.asg.net/plattform-base-images/ubi9/ubi-minimal
- cpk.quay.asg.net/plattform-base-images/ubi9/ubi-micro
- cpk.quay.asg.net/plattform-base-images/ubi9/ubi-init

## UBI  8   (W i dely   Used) :

- cpk.quay.asg.net/plattform-base-images/ubi8/ubi
- cpk.quay.asg.net/plattform-base-images/ubi8/ubi-minimal
- cpk.quay.asg.net/plattform-base-images/ubi8/ubi-micro
- cpk.quay.asg.net/plattform-base-images/ubi8/ubi-init

## UBI  7   (Legacy   Support) :

- cpk.quay.asg.net/plattform-base-images/ubi7/ubi
- cpk.quay.asg.net/plattform-base-images/ubi7/ubi-minimal
- cpk.quay.asg.net/plattform-base-images/ubi7/ubi-init

## OpenJDK Runt i me   Images

OpenJDK 21 (Latest   LTS)   -   UBI   8 :

- cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-21
- cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-21-runtime

## OpenJDK 17 (LTS)   -   UBI   8 :

- cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-17
- cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-17-runtime

## OpenJDK 17 (LTS)   -   UBI   9 :

- cpk.quay.asg.net/plattform-base-images/ubi9/openjdk-17
- cpk.quay.asg.net/plattform-base-images/ubi9/openjdk-17-runtime

## OpenJDK 11  (LTS)   -   UBI   8 :

- cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-11
- cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-11-runtime

## Python   Runt i me   Images

## Python  3 . 12   (Latest) :

- cpk.quay.asg.net/plattform-base-images/ubi9/python-312
- cpk.quay.asg.net/plattform-base-images/ubi8/python-312

## Python  3 . 11 :

- cpk.quay.asg.net/plattform-base-images/ubi9/python-311
- cpk.quay.asg.net/plattform-base-images/ubi8/python-311

## Python  3 . 10 :

- cpk.quay.asg.net/plattform-base-images/ubi9/python-310
- cpk.quay.asg.net/plattform-base-images/ubi8/python-310

## 4.  Schr i tt-f ü r-Schr i tt   Anle i tung

## 4. 1   Verwendung  i n   e i nem   Dockerf i le

Um ein Basis-Image in einem Dockerfile zu verwenden, geben Sie die vollständige URL des Images an:

```
1 2 # Beispiel für die Verwendung eines Node.js Basis-Images 3 FROM cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-21 4 5 WORKDIR /app 6 7 COPY target/app.jar app.jar 8 9 EXPOSE 8080 10 11 CMD ["java", "-jar", "app.jar"] 12
```

## 4. 2   Verwendung  i n   e i ner   OpenSh i ft   YAML-Date i

Um einen Pod mit einem Basis-Image zu starten, verwenden Sie die folgende YAML-Konfiguration:

```
1 2 apiVersion 3 kind : Pod
```

```
: v1
```

```
4 metadata : 5 name : meine-anwendung 6 spec : 7 containers : 8 -name : app-container 9 image : cpk.quay.asg.net/plattform-base-images/ubi9/python-310 10 ports : 11 -containerPort : 8080 12
```

## 4. 3   Auswahl   der   r i cht i gen   Tags

Die Wahl des richtigen Image-Tags ist entscheidend für die Stabilität Ihrer Anwendung:

## Für   Entw i cklungsumgebungen :

- Verwenden Sie die Basis-URLs ohne spezifische Versions-Tags
- Beispiel: cpk.quay.asg.net/plattform-base-images/ubi9/python-312
- Beispiel: cpk.quay.asg.net/plattform-base-images/ubi8/openjdk-17

Dies ermöglicht automatische Updates auf neueste Patch-Versionen und vereinfacht die Entwicklung.

## * Für   Produkt i onsumgebungen : *

- Verwenden Sie immer vollständige Versionsnummern
- Beispiel: `&lt;image&gt;:&lt;vollständige-version&gt;` wie `node:18.17.1` oder `python:3.11.4`
- Das verwenden `latest` oder Major-Tags in Produktionsumgebungen ist verboten.

## 4. 4   Anfordern   zus ä tzl i cher   Images

Wenn Sie ein Basis-Image benötigen, das nicht verfügbar ist:

- 1. Senden Sie eine E-Mail an plattform@team.de
- 2. Geben Sie an, welches Image Sie benötigen und wofür
- 3. Das Plattformteam wird ihre Anfrage bearbeiten und dann das gewünschte Image bereitstellen

## 5.  H ä uf i ge   Fehler   und   L ö sungen

## 5. 1   Image   Pull   Fehler

* Problem: * `ImagePullBackOff` oder `ErrImagePull` Fehler beim Starten eines Pods.

## * Lösung: *

- Überprüfen Sie die genaue Schreibweise der Image-URL
- Stellen Sie sicher, dass das angegebene Tag existiert
- Überprüfen Sie Ihre Netzwerkverbindung zum Registry-Server

```
# Detaillierte Informationen zum Pod anzeigen
```

```
1 2 # Überprüfen Sie den Status des Pods 3 oc get pods 4 5 6 oc describe pod <pod-name> 7
```

## 5. 2   Veraltete   Images

* Problem: * Sicherheitslücken durch Verwendung veralteter Images.

## * Lösung: *

- Aktualisieren Sie regelmäßig Ihre Image-Tags auf die neuesten Versionen
- Implementieren Sie einen automatisierten Prozess zur Überprüfung von Image-Versionen
- Verwenden Sie in der Produktion immer spezifische Versionen und aktualisieren Sie diese bewusst

## 6.  We i terf ü hrende   L i nks

- RedHat Universal Base Images (UBI) Dokumentation(https://access.redhat.com/documentation/enus/red\_hat\_enterprise\_linux/8/html/building\_running\_and\_managing\_containers/using\_red\_hat\_universal\_base\_images\_standar d\_minimal\_and\_runtimes)
- Best Practices für Container-Images(https://docs.openshift.com/container-platform/4.10/openshift\_images/createi mages.html)
- OpenShift Container Platform Dokumentation(https://docs.openshift.com/container-platform/latest/welcome/index.html)

Das Bild zeigt ein Symbol in Form eines blauen Kreises, in dessen Mitte sich ein weißes Informations-Icon befindet. Dieses Icon besteht aus einem kleinen weißen Kreis oben und einem vertikalen Strich darunter, der wie der Buchstabe "i" aussieht. Der Hintergrund des Symbols ist ein helles Blau. Es repräsentiert typischerweise eine Kennzeichnung für Informationen oder Hinweise.

## Hi nwe i s

Bei weiteren Fragen oder Problemen wenden Sie sich bitte an das Plattformteam unter plattform@team.de.

Automatisch generiert mit LangChain und OpenAI GPT-4