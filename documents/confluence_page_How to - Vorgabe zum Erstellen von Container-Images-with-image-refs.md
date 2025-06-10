## How   to   - Vorgabe   zum   Erstellen   von   Conta i ner-Images

## 1 .  Z i el

Diese Anleitung beschreibt die Best Practices und verbindlichen Anforderungen für die Erstellung von Container-Images in unserer OpenShift-Umgebung. Nach Befolgung dieser Anleitung werden Sie in der Lage sein:

- Container-Images zu erstellen, die den Unternehmensstandards entsprechen
- Die erforderlichen Metadaten (Labels) korrekt zu implementieren
- Build-Logs ordnungsgemäß zu speichern, um Nachvollziehbarkeit zu gewährleisten

## 2 .  Voraussetzungen

- Zugang zur RedHat OpenShift Container Platform
- Grundlegende Kenntnisse über Container und Dockerfile-Syntax
- Berechtigungen zum Erstellen und Ausführen von Pipelines in OpenShift
- Git-Repository für Ihren Anwendungscode

## 3.  Schr i tt-f ü r-Schr i tt   Anle i tung

## 3. 1   Implement i erung   der   erforderl i chen   Labels

Jedes Container-Image MUSS die folgenden OpenContainers-Labels enthalten:

Die Image Labels können über den Containerfile hinzugefügt werden

```
1 2 # Beispiel für Containerfile-Labels 3 LABEL org.opencontainers.image.created="2023-05-15T09:00:00Z" \ 4 org.opencontainers.image.vendor="Meine Firma GmbH" \ 5 org.opencontainers.image.authors="team@meinefirma.de" \ 6 org.opencontainers.image.source="https://github.com/meinefirma/meine-app" \ 7 org.opencontainers.image.revision="a1b2c3d4e5f6" \ 8 org.opencontainers.image.title="Meine Anwendung" \ 9 org.opencontainers.image.description="Beschreibung meiner Anwendung" \ 10 org.opencontainers.image.version="1.0.0" 11
```

## Erklärung der Labels:

| Label                                     | Beschre i bung                      | Be i sp i el             |
|-------------------------------------------|-------------------------------------|--------------------------|
| -------                                   | -------------                       | ----------               |
| org . openconta i ners .i mage . creat ed | Erstellungsdatum i mISO-8601 Format | 2023-05-15T09 : 00 : 00Z |
| org . openconta i ners .i mage . vend or  | Name des Unternehmens               | Mene i F i rma GmbH      |

| org . openconta i ners .i mage . auth ors         | Kontakt i nformat i onen                           | team@me i nef i rma . de                            |
|---------------------------------------------------|----------------------------------------------------|-----------------------------------------------------|
| org . openconta i ners .i mage . sour ce          | URL zum Quellcode                                  | https : //g i thub . com/me i nef i rma/ mene-app i |
| org . openconta i ners .i mage . rev i s i on     | Gt-Comm i i t-Hash oder andere Rev i s i onsnummer | a1b2c3d4e5f6                                        |
| org . openconta i ners .i mage . t i tle          | Name der Anwendung                                 | Mene i Anwendung                                    |
| org . openconta i ners .i mage . desc r i pt i on | Kurze Beschre i bung                               | Beschre i bung mener i Anwendung                    |
| org . openconta i ners .i mage . vers i on        | Semant i sche Vers i onsnummer                     | 1 . 0 . 0                                           |

## 2.  E i nr i chtung   der   Bu i ld-Log-Spe i cherung

Um die Nachvollziehbarkeit zu gewährleisten, müssen alle Build-Logs gespeichert werden. Hierfür wird die ClusterTask "savepipeline-logs" verwendet.

- Integration in Ihre Pipeline:

```
1 2 apiVersion: tekton.dev/v1beta1 3 kind: Pipeline 4 metadata: 5 name: meine-anwendung-pipeline 6 spec: 7 params: 8 - name: git-url 9 type: string 10 - name: git-revision 11 type: string 12 - name: image-name 13 type: string 14 - name: image-tag 15 type: string 16 tasks: 17 - name: git-clone 18 taskRef: 19 name: git-clone 20 kind: ClusterTask 21 params: 22 - name: url 23 value: $(params.git-url) 24 - name: revision 25 value: $(params.git-revision) 26 workspaces: 27 - name: output 28 workspace: shared-workspace
```

```
29 30 - name: build-image 31 taskRef: 32 name: buildah 33 kind: ClusterTask 34 params: 35 - name: IMAGE 36 value: $(params.image-name):$(params.image-tag) 37 workspaces: 38 - name: source 39 workspace: shared-workspace 40 runAfter: 41 - git-clone 42 43 # Speichern der Build-Logs 44 - name: save-logs 45 taskRef: 46 name: save-pipeline-logs 47 kind: ClusterTask 48 params: 49 - name: Anwendung 50 value: "meine-anwendung" 51 - name: Version 52 value: $(params.image-tag) 53 runAfter: 54 - build-image 55 56 workspaces: 57 - name: shared-workspace 58
```

## 3.  Ausf ü hren   der   P i pel i ne

- 1. Erstellen Sie die Pipeline-Ressource in OpenShift:
- 2. Starten Sie die Pipeline mit den erforderlichen Parametern:

```
1 2 oc apply -f meine-pipeline.yaml 3
```

```
1 2 oc create -f - <<EOF 3 apiVersion: tekton.dev/v1beta1 4 kind: PipelineRun 5 metadata: 6 generateName: meine-anwendung-pipeline-run7 spec: 8 pipelineRef: 9 name: meine-anwendung-pipeline 10 params: 11 - name: git-url 12 value: "https://github.com/meinefirma/meine-app" 13 - name: git-revision 14 value: "main" 15 - name: image-name 16 value: "image-registry.openshift-image-registry.svc:5000/mein-projekt/meine-anwendung" 17 - name: image-tag 18 value: "1.0.0"
```

```
19 workspaces: 20 - name: shared-workspace 21 persistentVolumeClaim: 22 claimName: meine-pipeline-pvc 23 EOF 24
```

## 4.  H ä uf i ge   Fehler   und   L ö sungen

- 1. Fehlende Labels

Problem : Build schlägt fehl mit Hinweis auf fehlende Labels.

L ö sung : Überprüfen Sie Ihr Dockerfile und stellen Sie sicher, dass alle erforderlichen Labels vorhanden sind. Verwenden Sie das oben gezeigte Beispiel als Vorlage.

- 2. Fehler bei der Log-Speicherung

Problem : Die Task "save-pipeline-logs" schlägt fehl.

## L ö sung :

- Überprüfen Sie, ob die ClusterTask "save-pipeline-logs" im Cluster verfügbar ist:
- Stellen Sie sicher, dass die Parameter "Anwendung" und "Version" korrekt übergeben werden
- Überprüfen Sie die Berechtigungen Ihres Service-Accounts
- 3. Pipeline kann nicht auf Git-Repository zugreifen

```
1 2 oc get clustertask save-pipeline-logs 3
```

Problem : Die Pipeline kann nicht auf das Git-Repository zugreifen.

## L ö sung :

- Überprüfen Sie die URL des Repositories
- Stellen Sie sicher, dass die notwendigen Zugriffsberechtigungen konfiguriert sind
- Bei privaten Repositories: Konfigurieren Sie ein Secret für den Zugriff

## 4.  We i terf ü hrende   L i nks

- OpenShift Pipelines Dokumentation(https://docs.openshift.com/container-platform/4.10/cicd/pipelines/understandingopenshift-pipelines.html)
- OpenContainers Image Spec(https://github.com/opencontainers/image-spec/blob/main/annotations.md)
- Tekton Pipelines Dokumentation(https://tekton.dev/docs/pipelines/)
- Best Practices für Container-Images(https://docs.openshift.com/container-platform/4.10/openshift\_images/createi mages.html)

Das Bild zeigt ein rundes Symbol mit einem blauen Hintergrund. In der Mitte des Symbols befindet sich ein weißes, kleines "i", welches als Informationssymbol dient. Das Design ist schlicht und klar, wodurch es einfach erkennbar ist. Dieses Icon wird oft verwendet, um Zugriff auf zusätzlichen Kontext oder Erklärungen zu geben.

## Hi nwe i s

Diese Anleitung basiert auf den aktuellen Unternehmensstandards. Bei Fragen oder Unklarheiten wenden Sie sich bitte an das Platform-Team.