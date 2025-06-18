## How to - Image Annahme und HELM Chart Annahme

## 1. Ziel

Dieses How-To beschreibt den Prozess, wie externe Container Images und Helm Charts in die RedHat OpenShift Container Platform importiert werden können. Da der Cluster keinen direkten Zugang zum Internet hat, müssen diese Ressourcen über einen definierten Annahmeprozess in die Plattform überführt werden. Nach Abschluss des Prozesses stehen die Images und Helm Charts für die Nutzung innerhalb der Plattform zur Verfügung.

## 2. Voraussetzungen

- Gültiges Benutzerkonto für die OpenShift Container Platform
- Zugehörigkeit zu einem Applikationsteam mit entsprechenden Berechtigungen
- Grundlegende Kenntnisse über Container und Helm Charts
- Zugang zum internen Registry-Service
- Installierte OpenShift CLI (`oc`)
- Optional: Installierte Helm CLI für lokale Tests

## 3. Prüfung auf vertrauenswürdige Quellen

Vor dem Import von Container Images und Helm Charts ist eine sorgfältige Prüfung der Quellen erforderlich, um die Sicherheit und Compliance der Plattform zu gewährleisten.

## 3.1 Bewertung von Container Image Quellen

## Vertrauenswürdige Registries:

- Docker Hub (Docker: Accelerated Container Application Development) - nur für offizielle Images und verifizierte Publisher
- Red Hat Container Catalog (Home - Red Hat Ecosystem Catalog)
- Quay (Quay) - bevorzugt für Red Hat und Community Images
- Bitnami Registry für etablierte Anwendungsstacks

## Sicherheitskriterien für Images:

- Verwendung von offiziellen Base Images (alpine, ubuntu, centos-stream, ubi)
- Aktuelle Sicherheitsupdates und Patch-Level
- Minimale Anzahl bekannter Vulnerabilities (CVE-Score &lt; 7.0)
- Dokumentierte Dockerfile und Transparenz der Build-Prozesse
- Regelmäßige Updates durch den Maintainer

## 3.2 Bewertung von Helm Chart Quellen

## Vertrauenswürdige Helm Repositories:

- Artifact Hub (Artifact Hub) - zentraler Hub für Helm Charts
- Bitnami Charts (charts.bitnami.com/bitnami)
- Elastic Charts (helm.elastic.co)
- Prometheus Community (Prometheus Community Kubernetes Helm Charts)
- Ingress NGINX (Welcome - Ingress-Nginx Controller)

## Qualitätskriterien für Helm Charts:

- Wartung durch etablierte Organisationen oder Communities
- Regelmäßige Updates und Versionszyklen
- Umfassende Dokumentation und Konfigurationsoptionen
- Verwendung von bewährten Kubernetes-Praktiken
- Aktive Community und Support

## 4. Schritt-für-Schritt Anleitung

## 4.1 Image Annahme

Das Bild zeigt ein orangefarbenes Warnsymbol in Form eines Dreiecks, das mit einem weißen Ausrufezeichen in der Mitte versehen ist. Der Hintergrund des Symbols ist hellgelb, wodurch das Dreieck und das Ausrufezeichen deutlich hervorgehoben werden. Dieses Symbol wird häufig verwendet, um auf eine Warnung oder eine potenzielle Gefahr hinzuweisen.

## 4.1.1 Identifizierung des benötigten Images

1. Ermitteln Sie die genaue Image-Referenz (Repository und Tag), die Sie importieren möchten.

Beispiel: `docker.io/nginx:1.21.0`

2. Prüfen Sie, ob das Image bereits in der internen Registry vorhanden ist:

```bash

oc get is -n &lt;namespace&gt;

```

## 4.1.2 Erstellen einer Image-Annahme-Anfrage

1. Erstellen Sie eine YAML-Datei `image-request.yaml` mit folgendem Inhalt:

```yaml

apiVersion: operator.openshift.io/v1alpha1

kind: ImageImportRequest

metadata:

name: nginx-import

namespace: &lt;team-namespace&gt;

spec:

source:

registry: docker.io

repository: nginx

tag: 1.21.0

destination:

registry: registry.internal.example.com

repository: &lt;team-namespace&gt;/nginx

Images sind Applikationsteam-spezifisch. Jedes Team erhält nur Zugriff auf die eigenen Images.

```

2. Reichen Sie die Anfrage ein:

```bash

oc apply -f image-request.yaml

```

3. Überprüfen Sie den Status der Anfrage:

```bash

oc get imagerequest nginx-import -n &lt;team-namespace&gt; -o yaml

```

## 4.1.3 Verwendung des importierten Images

Nach erfolgreicher Annahme können Sie das Image in Ihren Deployments verwenden:

```yaml

apiVersion: apps/v1

kind: Deployment

metadata:

name: nginx-deployment

namespace: &lt;team-namespace&gt;

spec:

replicas: 1

selector:

matchLabels:

app: nginx

template:

metadata:

labels:

app: nginx

spec:

containers:

name: nginx

i mage: registry.internal.example.com/&lt;team-namespace&gt;/nginx:1.21.0

ports:

containerPort: 80

```

## 4.2 Helm Chart Annahme

## 4.2.1 Identifizierung des benötigten Helm Charts

1. Ermitteln Sie die genaue Chart-Referenz und Version, die Sie importieren möchten.

Beispiel: `bitnami/wordpress:12.1.3`

2. Prüfen Sie, ob das Chart bereits im internen Helm Repository verfügbar ist:

```bash

helm search repo internal-helm-repo

```

## 4.2.2 Erstellen einer Helm Chart-Annahme-Anfrage

1. Erstellen Sie eine YAML-Datei `helmchart-request.yaml` mit folgendem Inhalt:

```yaml

apiVersion: operator.openshift.io/v1alpha1

kind: HelmChartImportRequest

metadata:

name: wordpress-import

namespace: &lt;team-namespace&gt;

spec:

source:

repository: https://charts.bitnami.com/bitnami

chart: wordpress

version: 12.1.3

destination:

repository: internal-helm-repo chart: &lt;team-namespace&gt;-wordpress

version: 12.1.3

```

2. Reichen Sie die Anfrage ein:
```bash

oc apply -f helmchart-request.yaml

```

3. Überprüfen Sie den Status der Anfrage:

```bash
oc get helmchartrequest wordpress-import -n &lt;team-namespace&gt; -o yaml

```

## 4.2.3 Verwendung des importierten Helm Charts

Nach erfolgreicher Annahme können Sie das Helm Chart installieren:

1. Aktualisieren Sie Ihre Helm Repositories:
```bash

helm repo update

```

2. Installieren Sie das Chart:
```bash

helm install my-wordpress internal-helm-repo/&lt;team-namespace&gt;-wordpress --version 12.1.3 -n &lt;team-namespace&gt;

```

## 5. Häufige Fehler und Lösungen

## 5.1 Image kann nicht gefunden werden

*Problem:* Die Image-Annahme schlägt fehl mit "Image not found".

*Lösung:*

- Überprüfen Sie die genaue Schreibweise des Image-Namens und Tags
- Stellen Sie sicher, dass das Image im Quell-Repository öffentlich verfügbar ist
- Bei privaten Repositories: Stellen Sie sicher, dass die entsprechenden Zugangsdaten konfiguriert sind

## 5.2 Berechtigungsprobleme

Das Bild zeigt ein Symbol, das eine stilisierte Darstellung einer Kette oder eines Links darstellt. Es besteht aus zwei rechteckigen, leicht abgerundeten Formen, die durch eine diagonale Verbindung miteinander verbunden sind. Die Farbe des Symbols ist grau, und es wirkt minimalistisch. Dieses Symbol wird oft verwendet, um auf Hyperlinks oder Verknüpfungen hinzudeuten.

*Problem:* "Permission denied" beim Zugriff auf Images oder Helm Charts.

*Lösung:*

- Überprüfen Sie Ihre Teamzugehörigkeit und Berechtigungen
- Stellen Sie sicher, dass Sie im richtigen Namespace arbeiten
- Wenden Sie sich an den Plattform-Administrator, falls die Berechtigungen angepasst werden müssen

## 5.3 Helm Chart Versionskonflikte

*Problem:* Konflikte bei der Installation aufgrund von Abhängigkeiten.

*Lösung:*

- Überprüfen Sie die Abhängigkeiten des Charts mit `helm dependency list`
- Importieren Sie fehlende abhängige Charts ebenfalls
- Verwenden Sie kompatible Versionen aller Komponenten

## 5.4 Image Pull Fehler im Deployment

*Problem:* Pods können nicht gestartet werden mit "ImagePullBackOff".

*Lösung:*

- Überprüfen Sie, ob das Image erfolgreich importiert wurde
- Stellen Sie sicher, dass der Image-Pfad in der Deployment-Konfiguration korrekt ist
- Prüfen Sie die Image Pull Secrets, falls erforderlich

## 6. Weiterführende Links

- OpenShift Container Platform Dokumentation(https://docs.openshift.com/)
- Helm Dokumentation(https://helm.sh/docs/)
- Interne Plattform-Dokumentation(https://confluence.internal.example.com/container-platform/)
- OpenShift Image Streams Konzepte(https://docs.openshift.com/container-platform/4.10/openshift\_images/image-streamsmanage.html)
- Best Practices für Container Images(https://docs.openshift.com/container-platform/4.10/openshift\_images/createi mages.html)
- Bei weiteren Fragen oder Problemen wenden Sie sich bitte an das Plattform-Team über den Support-Kanal oder per E-Mail an platform-support@icp-company.de.

Das Bild zeigt ein Symbol, das aus einem blauen Kreis besteht. Im Zentrum des Kreises befindet sich ein weißes, stilisiertes "i", das für "Information" steht. Der Kreis hat eine einheitliche Färbung, und das Symbol vermittelt eine klare und minimalistische Darstellung. Es wird häufig als Hinweis auf verfügbare Informationen oder als Icon für Hilfs- und Informationsoptionen in digitalen Anwendungen verwendet. Der Hintergrund ist hell und neutral, wodurch das Symbol hervorgehoben wird.

Automatisch generiert mit LangChain und OpenAI GPT-4