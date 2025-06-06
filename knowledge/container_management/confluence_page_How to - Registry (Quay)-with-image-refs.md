## How   to   Reg i stry   (Quay)

## 1 .  Z i el

In diesem How-To werden verschiedene, für Applikationsteams besonders relevante, Eigenschaften und Vorgehensweisen für die Container Registry, basierend auf Red Hat Quay, gesammelt. Die Anleitung zeigt, wie Sie:

- Sich bei der Registry anmelden
- Container Images hochladen und herunterladen
- HELM Charts verwalten
- Repositories organisieren und verwalten

Die Registry kann zur Ablage von allen OCI-Compliant Objekten genutzt werden. Dies sind vor allem Containerimages und paketierte HELM Charts.

## 2.  Voraussetzungen

- Gültige DAS ID und Passwort für den SSO-Login
- Installierte Docker CLI oder Podman
- Installierte OpenShift CLI (oc)
- Grundlegende Kenntnisse über Container und Container Images
- Berechtigungen für das entsprechende Applikationsteam-Repository

## 3.  Schr i tt-f ü r-Schr i tt   Anle i tung

## 3 . 1 Zugr i ff auf   d i e Reg i stry

Die Registry ist unter folgender URL erreichbar:

```
Die URL eines Applikationsteam-Images folgt diesem Schema: 1 cpk.quay.asg.net 1 cpk.quay.asg.net/<app-team-name>-<umgebung>/<repo>:<tag>
```

## 3 . 2 Anmeldung   an   der   Reg i stry

## 3. 2 . 1   Web-Interface   Log i n

- 1. Navigieren Sie zur Registry-URL:
- 2. Klicken Sie auf den "Log in"-Button
- 3. Sie werden zum SSO-Login weitergeleitet
- 4. Geben Sie Ihre DAS ID und Ihr Passwort ein
- 5. Nach erfolgreicher Authentifizierung werden Sie zur Registry-Oberfläche weitergeleitet

```
1 cpk.quay.asg.net
```

## 3. 2 . 2   CLI   Log i n

Um sich über die Kommandozeile anzumelden, verwenden Sie folgenden Befehl:

```
1 2 docker login <cpk.quay.asg.net> -u <DAS-ID>
```

```
3
```

```
Oder mit Podman:
```

```
1 2 podman login <cpk.quay.asg.net> -u <DAS-ID> 3
```

Sie werden nach Ihrem Passwort gefragt. Geben Sie dieses ein, um sich anzumelden.

## 3 . 3 Images   verwalten

## 3. 3 . 1   Image   hochladen   (Push)

- 1. Taggen Sie Ihr lokales Image entsprechend der Registry-Namenskonvention:
- 2. Laden Sie das Image hoch:

```
1 2 docker tag mein-lokales-image:latest <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> 3
```

```
1 2 docker push <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> 3
```

## 3. 3 . 2   Image   herunterladen   (Pull)

Um ein Image aus der Registry herunterzuladen:

```
1 2 docker pull <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> 3
```

## 3 . 4 HELM   Charts   verwalten

## 3. 4 . 1   HELM   Chart   paket i eren

- 1. Navigieren Sie zum Verzeichnis Ihres HELM Charts
- 2. Paketieren Sie das Chart:

```
1 2 3
```

```
helm package .
```

## 3. 4 . 2   HELM   Chart  i n   d i e   Reg i stry   hochladen

```
1 2 helm push <chart-name>-<version>.tgz oci://<REGISTRY-URL>/<app-team-name>-<umgebung>/helm-charts 3
```

## 3. 4 . 3   HELM   Chart   aus   der   Reg i stry  i nstall i eren

```
1 2 helm install <release-name> oci://<REGISTRY-URL>/<app-team-name>-<umgebung>/helm-charts/<chart-name> --version <v 3
```

## 3 . 5 Repos i tory-Verwaltung  i m   Web-Interface

- 1. Navigieren Sie zur Registry-URL und melden Sie sich an
- 2. Wählen Sie Ihre Organisation (&lt;app-team-name&gt;-&lt;umgebung&gt;) aus
- 3. Hier können Sie:
- Neue Repositories erstellen
- Berechtigungen verwalten
- Tags und Images inspizieren
- Vulnerability-Scans einsehen (falls aktiviert)
- Image-Metadaten anzeigen

## 3 . 6 Integrat i on   m i t OpenSh i ft

## 3. 6 . 1   Secret   f ü r   Reg i stry-Zugr i ff   erstellen

```
1 2 oc create secret docker-registry quay-secret \ 3 --docker-server=<REGISTRY-URL> \ 4 --docker-username=<DAS-ID> \ 5 --docker-password=<PASSWORD> \ 6 --namespace=<namespace> 7
```

## 3. 6 . 2   Secret   zu   Serv i ceAccount   h i nzuf ü gen

```
1 2 oc secrets link default quay-secret --for=pull --namespace=<namespace> 3
```

## 3. 6 . 3   Deployment   m i t   Reg i stry-Image   erstellen

```
1 2 apiVersion: apps/v1 3 kind: Deployment 4 metadata: 5 name: meine-app 6 spec: 7 replicas: 1 8 selector: 9 matchLabels: 10 app: meine-app 11 template: 12 metadata: 13 labels: 14 app: meine-app 15 spec: 16 containers: 17 - name: meine-app 18 image: <REGISTRY-URL>/<app-team-name>-<umgebung>/<repo>:<tag> 19 ports: 20 - containerPort: 8080 21 imagePullSecrets: 22 - name: quay-secret 23
```

## 4.  Organ i sat i ons

Innerhalb von Quay stehen für jedes Applikationsteam verschiedene 'Organisations' bereit, die jeweils aus der ID des Applikationsteams und einem Suffix bestehen.

Über den Suffix werden die Organisations den verschiedenen Systemumgebungen zugeordnet.

- -dev → dev (Entwicklungsumgebung)
- -qs → qs (Qualitätssicherungsumgebung)
- -prod → prod (Produktionsumgebung)

In der Containerplattform dürfen nur Images gestartet werden, die in der zur Systemumgebung passenden Organisation liegen.

## 5.  H ä uf i ge   Fehler   und   L ö sungen

## 5 . 1 Authent i f i z i erungsprobleme

* Problem *: "unauthorized: authentication required"

## * L ö sung *:

- Überprüfen Sie Ihre Anmeldedaten
- Führen Sie erneut `docker login` oder `podman login` aus
- Stellen Sie sicher, dass Ihre DAS ID Zugriff auf das Repository hat

## 5 . 2 Berecht i gungsprobleme

* Problem *: "denied: requested access to the resource is denied"

## * L ö sung *:

- Überprüfen Sie, ob Sie Schreibrechte für das Repository haben
- Kontaktieren Sie den Repository-Administrator, um Berechtigungen zu erhalten
- Überprüfen Sie die Namenskonvention des Repositories

## 5 . 3 Image   n i cht   gefunden

* Problem *: "manifest unknown: manifest unknown"

## * L ö sung *:

- Überprüfen Sie, ob das Image und der Tag korrekt sind
- Stellen Sie sicher, dass das Image tatsächlich in der Registry existiert
- Überprüfen Sie die Schreibweise und Groß-/Kleinschreibung

## 5 . 4 OpenSh i ft   kann   Image   n i cht   pullen

* Problem *: "ImagePullBackOff" oder "ErrImagePull"

## * L ö sung *:

- Überprüfen Sie, ob das Secret korrekt erstellt wurde
- Stellen Sie sicher, dass das Secret dem ServiceAccount zugeordnet ist
- Überprüfen Sie die Image-URL im Deployment

## 6.  We i terf ü hrende   L i nks

- Red Hat Quay Dokumentation(https://access.redhat.com/documentation/en-us/red\_hat\_quay/)
- OCI Registry as Storage (ORAS) Dokumentation(https://oras.land/docs/)
- OpenShift Container Platform Dokumentation(https://docs.openshift.com/)
- HELM OCI Support Dokumentation(https://helm.sh/docs/topics/registries/)
- Docker Registry Dokumentation(https://docs.docker.com/registry/)

Das Bild zeigt ein einfaches rundes Symbol mit blauem Hintergrund. Innerhalb des Kreises befindet sich ein weißes "i", welches Informationen symbolisiert. Es dient typischerweise als Icon für Hinweise oder Informationsabschnitte in Benutzeroberflächen. Der blaue Kreis hat eine klare und gleichmäßige Farbe, und die Darstellung ist minimalistisch gehalten.-with-image-refs_artifacts/image_000000_479f5dbe0bba87f08e75ec9e97c813b0e1f7c4927c56e6ab3404359a7ea7750a.png)

## H i nwe i s

Ersetzen Sie in allen Beispielen `&lt;REGISTRY-URL&gt;`, `&lt;app-team-name&gt;`, `&lt;umgebung&gt;`, `&lt;repo&gt;`, `&lt;tag&gt;`, `&lt;DAS-ID&gt;`, `&lt;PASSWORD&gt;` und `&lt;namespace&gt;` durch Ihre tatsächlichen Werte.

Automatisch generiert mit LangChain und OpenAI GPT-4