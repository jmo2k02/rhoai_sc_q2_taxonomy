## How to Registry (Quay)

## 1. Ziel

In diesem How-To werden verschiedene, für Applikationsteams besonders relevante, Eigenschaften und Vorgehensweisen für die Container Registry, basierend auf Red Hat Quay, gesammelt. Die Anleitung zeigt, wie Sie:

- Sich bei der Registry anmelden
- Container Images hochladen und herunterladen
- HELM Charts verwalten
- Repositories organisieren und verwalten

Die Registry kann zur Ablage von allen OCI-Compliant Objekten genutzt werden. Dies sind vor allem Containerimages und paketierte HELM Charts.

## 2. Voraussetzungen

- Gültige DAS ID und Passwort für den SSO-Login
- Installierte Docker CLI oder Podman
- Installierte OpenShift CLI (oc)
- Grundlegende Kenntnisse über Container und Container Images
- Berechtigungen für das entsprechende Applikationsteam-Repository

## 3. Schritt-für-Schritt Anleitung

## 3.1 Zugriff auf die Registry

Die Registry ist unter folgender URL erreichbar:

Die URL eines Applikationsteam-Images folgt diesem Schema: 
```
cpk.quay.asg.net
cpk.quay.asg.net/<app-team-name>-<umgebung>/<repo>:<tag>
```

## 3.2 Anmeldung an der Registry

## 3.2.1 Web-Interface Login

1. Navigieren Sie zur Registry-URL:
2. Klicken Sie auf den "Log in"-Button
3. Sie werden zum SSO-Login weitergeleitet
4. Geben Sie Ihre DAS ID und Ihr Passwort ein
5. Nach erfolgreicher Authentifizierung werden Sie zur Registry-Oberfläche weitergeleitet

```
1 cpk.quay.asg.net
```

## 3.2.2 CLI Login

Um sich über die Kommandozeile anzumelden, verwenden Sie folgenden Befehl:

```
docker login <cpk.quay.asg.net> -u <DAS-ID>
```

Oder mit Podman:

```
podman login <cpk.quay.asg.net> -u <DAS-ID> 3
```

Sie werden nach Ihrem Passwort gefragt. Geben Sie dieses ein, um sich anzumelden.

## 3.3 Images verwalten

## 3.3.1 Image hochladen (Push)

1. Taggen Sie Ihr lokales Image entsprechend der Registry-Namenskonvention:
2. Laden Sie das Image hoch:

```
docker tag mein-lokales-image:latest <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> 3
```

```
docker push <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> 3
```

## 3.3.2 Image herunterladen (Pull)

Um ein Image aus der Registry herunterzuladen:

```
docker pull <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> 3
```

## 3.4 HELM Charts verwalten

## 3.4.1 HELM Chart paketieren

1. Navigieren Sie zum Verzeichnis Ihres HELM Charts
2. Paketieren Sie das Chart:

```
helm package .
```

## 3.4.2 HELM Chart in die Registry hochladen

```
helm push <chart-name>-<version>.tgz oci://<REGISTRY-URL>/<app-team-name>-<umgebung>/helm-charts 3
```

## 3.4.3 HELM Chart aus der Registry installieren

```
helm install <release-name> oci://<REGISTRY-URL>/<app-team-name>-<umgebung>/helm-charts/<chart-name> --version <v 3
```

## 3.5 Repository-Verwaltung im Web-Interface

1. Navigieren Sie zur Registry-URL und melden Sie sich an
2. Wählen Sie Ihre Organisation (&lt;app-team-name&gt;-&lt;umgebung&gt;) aus
3. Hier können Sie:
    - Neue Repositories erstellen
    - Berechtigungen verwalten
    - Tags und Images inspizieren
    - Vulnerability-Scans einsehen (falls aktiviert)
    - Image-Metadaten anzeigen

## 3.6 Integration mit OpenShift

## 3.6.1 Secret für Registry-Zugriff erstellen

```
oc create secret docker-registry quay-secret \ 3 --docker-server=<REGISTRY-URL> \ 4 --docker-username=<DAS-ID> \ 5 --docker-password=<PASSWORD> \ 6 --namespace=<namespace> 7
```

## 3.6.2 Secret zu ServiceAccount hinzufügen

```
oc secrets link default quay-secret --for=pull --namespace=<namespace> 3
```

## 3.6.3 Deployment mit Registry-Image erstellen

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meine-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: meine-app
  template:
    metadata:
      labels:
        app: meine-app
    spec:
      containers:
      - name: meine-app
        image: <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> ports:
        - containerPort: 8080
        imagePullSecrets:
        - name: quay-secret
```

## 4. Organisations

Innerhalb von Quay stehen für jedes Applikationsteam verschiedene 'Organisations' bereit, die jeweils aus der ID des Applikationsteams und einem Suffix bestehen.

Über den Suffix werden die Organisations den verschiedenen Systemumgebungen zugeordnet.

- dev → dev (Entwicklungsumgebung)
- qs → qs (Qualitätssicherungsumgebung)
- prod → prod (Produktionsumgebung)

In der Containerplattform dürfen nur Images gestartet werden, die in der zur Systemumgebung passenden Organisation liegen.

## 5. Häufige Fehler und Lösungen

## 5.1 Authentifizierungsprobleme

*Problem*: "unauthorized: authentication required"

*Lösung*:

- Überprüfen Sie Ihre Anmeldedaten
- Führen Sie erneut `docker login` oder `podman login` aus
- Stellen Sie sicher, dass Ihre DAS ID Zugriff auf das Repository hat

## 5.2 Berechtigungsprobleme

*Problem*: "denied: requested access to the resource is denied"

*Lösung*:

- Überprüfen Sie, ob Sie Schreibrechte für das Repository haben
- Kontaktieren Sie den Repository-Administrator, um Berechtigungen zu erhalten
- Überprüfen Sie die Namenskonvention des Repositories

## 5.3 Image nicht gefunden

*Problem*: "manifest unknown: manifest unknown"

*Lösung*:

- Überprüfen Sie, ob das Image und der Tag korrekt sind
- Stellen Sie sicher, dass das Image tatsächlich in der Registry existiert
- Überprüfen Sie die Schreibweise und Groß-/Kleinschreibung

## 5.4 OpenShift kann Image nicht pullen

*Problem*: "ImagePullBackOff" oder "ErrImagePull"

*Lösung*:

- Überprüfen Sie, ob das Secret korrekt erstellt wurde
- Stellen Sie sicher, dass das Secret dem ServiceAccount zugeordnet ist
- Überprüfen Sie die Image-URL im Deployment

## 6. Weiterführende Links

- Red Hat Quay Dokumentation(https://access.redhat.com/documentation/en-us/red\_hat\_quay/)
- OCI Registry as Storage (ORAS) Dokumentation(https://oras.land/docs/)
- OpenShift Container Platform Dokumentation(https://docs.openshift.com/)
- HELM OCI Support Dokumentation(https://helm.sh/docs/topics/registries/)
- Docker Registry Dokumentation(https://docs.docker.com/registry/)

Das Bild zeigt ein einfaches rundes Symbol mit blauem Hintergrund. Innerhalb des Kreises befindet sich ein weißes "i", welches Informationen symbolisiert. Es dient typischerweise als Icon für Hinweise oder Informationsabschnitte in Benutzeroberflächen. Der blaue Kreis hat eine klare und gleichmäßige Farbe, und die Darstellung ist minimalistisch gehalten.

## Hinweis

Ersetzen Sie in allen Beispielen `&lt;REGISTRY-URL&gt;`, `&lt;app-team-name&gt;`, `&lt;umgebung&gt;`, `&lt;repo&gt;`, `&lt;tag&gt;`, `&lt;DAS-ID&gt;`, `&lt;PASSWORD&gt;` und `&lt;namespace&gt;` durch Ihre tatsächlichen Werte.

Automatisch generiert mit LangChain und OpenAI GPT-4