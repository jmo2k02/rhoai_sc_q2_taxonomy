## How to - Vorgabe zum Erstellen von Container-Images

## 1. Ziel

Diese Anleitung beschreibt die Best Practices und verbindlichen Anforderungen für die Erstellung von Container-Images in unserer OpenShift-Umgebung. Nach Befolgung dieser Anleitung werden Sie in der Lage sein:

- Container-Images zu erstellen, die den Unternehmensstandards entsprechen
- Die erforderlichen Metadaten (Labels) korrekt zu implementieren
- Build-Logs ordnungsgemäß zu speichern, um Nachvollziehbarkeit zu gewährleisten

## 2. Voraussetzungen

- Zugang zur RedHat OpenShift Container Platform
- Grundlegende Kenntnisse über Container und Dockerfile-Syntax
- Berechtigungen zum Erstellen und Ausführen von Pipelines in OpenShift
- Git-Repository für Ihren Anwendungscode

## 3. Schritt-für-Schritt Anleitung

## 3.1 Implementierung der erforderlichen Labels

Jedes Container-Image MUSS die folgenden OpenContainers-Labels enthalten:

Die Image Labels können über den Containerfile hinzugefügt werden

```
# Beispiel für Containerfile-Labels
LABEL org.opencontainers.image.created="2023-05-15T09:00:00Z" \
org.opencontainers.image.vendor="Meine Firma GmbH" \
org.opencontainers.image.authors="team@meinefirma.de" \
org.opencontainers.image.source="https://github.com/meinefirma/meine-app" \
org.opencontainers.image.revision="a1b2c3d4e5f6" \
org.opencontainers.image.title="Meine Anwendung" \
org.opencontainers.image.description="Beschreibung meiner Anwendung" \
org.opencontainers.image.version="1.0.0"
```

## Erklärung der Labels:

| Label                                     | Beschreibung                      | Beispiel             |
|-------------------------------------------|-------------------------------------|--------------------------|
| -------                                   | -------------                       | ----------               |
| org.opencontainers.image.created | Erstellungsdatum im ISO-8601 Format | 2023-05-15T09 : 00 : 00Z |
| org.opencontainers.image.vendor  | Name des Unternehmens               | Meine Firma GmbH      |

| org.opencontainers.image.authors         | Kontaktinformationen                           | team@meinefirma.de                            |
|---------------------------------------------------|----------------------------------------------------|-----------------------------------------------------|
| org.opencontainers.image.source          | URL zum Quellcode                                  | https://github.com/meinefirma/meine-app |
| org.opencontainers.image.revision     | Gt-Commit-Hash oder andere Revisionsnummer | a1b2c3d4e5f6                                        |
| org.opencontainers.image.title          | Name der Anwendung                                 | Meine Anwendung                                    |
| org.opencontainers.image.description | Kurze Beschreibung                               | Beschreibung meiner Anwendung                    |
| org.opencontainers.image.version        | Semantische Versionsnummer                     | 1.0.0                                           |

## 2. Einrichtung der Build-Log-Speicherung

Um die Nachvollziehbarkeit zu gewährleisten, müssen alle Build-Logs gespeichert werden. Hierfür wird die ClusterTask "savepipeline-logs" verwendet.

- Integration in Ihre Pipeline:

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: meine-anwendung-pipeline
spec:
  params:
  - name: git-url
    type: string
  - name: git-revision
    type: string
  - name: image-name
    type: string
  - name: image-tag
    type: string
  tasks:
  - name: git-clone
    taskRef:
      name: git-clone
      kind: ClusterTask
      params:
      - name: url
        value: $(params.git-url)
      - name: revision
        value: $(params.git-revision)
    workspaces:
      - name: output
        workspace: shared-workspace
  - name: build-image
    taskRef:
      name: buildah
      kind: ClusterTask
      params:
      - name: IMAGE
        value: $(params.image-name):$(params.image-tag)
    workspaces:
    - name: source
      workspace: shared-workspace
    runAfter:
    - git-clone
  # Speichern der Build-Logs
  - name: save-logs
    taskRef:
      name: save-pipeline-logs
      kind: ClusterTask
      params:
      - name: Anwendung
        value: "meine-anwendung"
      - name: Version
        value: $(params.image-tag)
    runAfter:
    - build-image
    workspaces:
    - name: shared-workspace
```

## 3. Ausführen der Pipeline

1. Erstellen Sie die Pipeline-Ressource in OpenShift:
2. Starten Sie die Pipeline mit den erforderlichen Parametern:

```
oc apply -f meine-pipeline.yaml
```

```
oc create -f - <<EOF
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  generateName: meine-anwendung-pipeline-run
spec:
  pipelineRef:
    name: meine-anwendung-pipeline
  params:
  - name: git-url
    value: "https://github.com/meinefirma/meine-app"
  - name: git-revision
    value: "main"
  - name: image-name
    value: "image-registry.openshift-image-registry.svc:5000/mein-projekt/meine-anwendung"
  - name: image-tag
    value: "1.0.0"
  workspaces:
  - name: shared-workspace
    persistentVolumeClaim:
      claimName: meine-pipeline-pvc EOF
```

## 4. Häufige Fehler und Lösungen

1. Fehlende Labels

Problem: Build schlägt fehl mit Hinweis auf fehlende Labels.

Lösung: Überprüfen Sie Ihr Dockerfile und stellen Sie sicher, dass alle erforderlichen Labels vorhanden sind. Verwenden Sie das oben gezeigte Beispiel als Vorlage.

2. Fehler bei der Log-Speicherung

Problem: Die Task "save-pipeline-logs" schlägt fehl.

Lösung:

- Überprüfen Sie, ob die ClusterTask "save-pipeline-logs" im Cluster verfügbar ist:
- Stellen Sie sicher, dass die Parameter "Anwendung" und "Version" korrekt übergeben werden
- Überprüfen Sie die Berechtigungen Ihres Service-Accounts

3. Pipeline kann nicht auf Git-Repository zugreifen

```
oc get clustertask save-pipeline-logs
```

Problem: Die Pipeline kann nicht auf das Git-Repository zugreifen.

Lösung:

- Überprüfen Sie die URL des Repositories
- Stellen Sie sicher, dass die notwendigen Zugriffsberechtigungen konfiguriert sind
- Bei privaten Repositories: Konfigurieren Sie ein Secret für den Zugriff

## 4. Weiterführende Links

- OpenShift Pipelines Dokumentation(https://docs.openshift.com/container-platform/4.10/cicd/pipelines/understandingopenshift-pipelines.html)
- OpenContainers Image Spec(https://github.com/opencontainers/image-spec/blob/main/annotations.md)
- Tekton Pipelines Dokumentation(https://tekton.dev/docs/pipelines/)
- Best Practices für Container-Images(https://docs.openshift.com/container-platform/4.10/openshift\_images/createi mages.html)

Das Bild zeigt ein rundes Symbol mit einem blauen Hintergrund. In der Mitte des Symbols befindet sich ein weißes, kleines "i", welches als Informationssymbol dient. Das Design ist schlicht und klar, wodurch es einfach erkennbar ist. Dieses Icon wird oft verwendet, um Zugriff auf zusätzlichen Kontext oder Erklärungen zu geben.

## Hinweis

Diese Anleitung basiert auf den aktuellen Unternehmensstandards. Bei Fragen oder Unklarheiten wenden Sie sich bitte an das Platform-Team.