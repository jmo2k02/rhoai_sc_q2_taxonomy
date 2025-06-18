# ICP Toollandschaft - Übersicht und Zuständigkeiten

## Überblick

Die Interne Container-Plattform (ICP) integriert verschiedene Technologien und Services zur Bereitstellung einer vollständigen Container-Orchestrierungsumgebung. Diese Seite dokumentiert alle eingebundenen Tools, deren Zuständigkeiten und Bewertungen.

## Zuständigkeitsmatrix der Toollandschaft

| Tool | Aufgabe | Beschreibung | Zuständige Abteilung | Technisch | Betrieblich | Plattform-Empfehlung | Risiko | Kommentar |
|------|---------|--------------|---------------------|-----------|-------------|---------------------|---------|-----------|
| **Red Hat OpenShift** | Container-Orchestrierung | Kubernetes-basierte Container-Plattform als Basis für ICP | ICP-Team | ICP-Team | ICP-Team | Empfohlen | Niedrig | Kern der ICP-Plattform |
| **Red Hat Quay (CPK)** | Container Registry | Zentrale Registry für Container-Images unter cpk.quay.asg.net | ICP-Team | ICP-Team | ICP-Team | Empfohlen | Niedrig | Team-spezifische Organisationen |
| **Argo CD** | GitOps Deployment | Kontinuierliche Bereitstellung über GitOps-Prinzipien | ICP-Team | ICP-Team | DevOps-Team | Empfohlen | Niedrig | Integration in ICP-Workflows |
| **Argo Workflows** | Workflow-Orchestrierung | Komplexe CI/CD-Pipelines und Batch-Processing | ICP-Team | ICP-Team | DevOps-Team | Empfohlen | Mittel | Für komplexe Workflows |
| **Tekton Pipelines** | CI/CD Pipelines | Native Kubernetes CI/CD mit ClusterTask "save-pipeline-logs" | ICP-Team | ICP-Team | ICP-Team | Empfohlen | Niedrig | Standard für Image-Builds |
| **Jenkins** | Legacy CI/CD | Bestehende Jenkins-Instanzen für Migration | DevOps-Team | External-ISV | DevOps-Team | Migration geplant | Hoch | Ablösung durch Tekton |
| **Helm 3** | Package Management | Kubernetes-Anwendungspaketierung (MUSS-Kriterium) | Applikationsteams | Applikationsteams | ICP-Team | Verpflichtend | Niedrig | Helm 2 nicht mehr unterstützt |
| **Istio Service Mesh** | Service-zu-Service Kommunikation | Mikrosegmentierung und mTLS zwischen Services | ICP-Team | ICP-Team | Netzwerk-Security-Team | Empfohlen | Mittel | Für komplexe Microservices |
| **HashiCorp Vault** | Secret Management | Zentrale Verwaltung sensibler Daten (KMS-System) | Security-Team | Security-Team | Security-Team | Verpflichtend | Niedrig | Kubernetes Secrets Integration |
| **Grafana** | Monitoring & Dashboards | Visualisierung von Metriken und Logs | ICP-Team | ICP-Team | ICP-Team | Empfohlen | Niedrig | Standard-Monitoring-Lösung |
| **Prometheus** | Metriken-Sammlung | Native Kubernetes-Metriken und Alerting | ICP-Team | ICP-Team | ICP-Team | Empfohlen | Niedrig | OpenShift-integriert |
| **Grafana Loki** | Log-Aggregation | Zentralisierte Log-Verwaltung für STDOUT/STDERR | ICP-Team | ICP-Team | ICP-Team | Empfohlen | Niedrig | Ersetzt ELK-Stack |
| **Azure Active Directory** | Identity Management | SSO und Benutzerauthentifizierung (DAS-ID Integration) | Identity-Management-Team | External-ISV | Identity-Management-Team | Verpflichtend | Mittel | RBAC-Integration erforderlich |
| **RHEL CoreOS** | Container-optimiertes OS | Immutable Betriebssystem für OpenShift-Nodes | Infrastructure-Team | Infrastructure-Team | Infrastructure-Team | Verpflichtend | Niedrig | OpenShift-Standard |
| **MinIO** | S3-kompatibles Storage | Objekt-Storage für Backup und Artefakte | Infrastructure-Team | Infrastructure-Team | Infrastructure-Team | Empfohlen | Mittel | Alternative zu AWS S3 |
| **VMware vSphere** | Virtualisierungs-Plattform | Unterliegende Infrastruktur für OpenShift-Cluster | Infrastructure-Team | Infrastructure-Team | Infrastructure-Team | Legacy | Hoch | Migration zu Cloud geplant |
| **Kasten K10** | Kubernetes Backup | Backup und Disaster Recovery für Kubernetes-Workloads | Infrastructure-Team | Infrastructure-Team | Infrastructure-Team | Empfohlen | Mittel | PV-Backup-Lösung |
| **Jira** | Projekt- und Issue-Management | Tracking von Anforderungen und Incidents | Project-Management | External-ISV | Project-Management | Standard | Niedrig | Organisatorische Unterstützung |
| **Confluence** | Dokumentationsplattform | Zentrale Wissensbasis und Dokumentation | ICP-Team | External-ISV | ICP-Team | Standard | Niedrig | Diese Dokumentation |
| **CSI Storage Provisioner** | Dynamic Storage | Automatische PV-Bereitstellung (CSI-kompatibel) | Infrastructure-Team | Infrastructure-Team | ICP-Team | Verpflichtend | Niedrig | MUSS-Kriterium für PVC/PV |
| **OpenShift Service Mesh** | Traffic Management | Ingress/Egress-Kontrolle und Service-Routing | ICP-Team | ICP-Team | Netzwerk-Security-Team | Empfohlen | Mittel | NetworkPolicy-Enforcement |

## Tool-Kategorien

### Kern-Plattform (Verpflichtend)
- Red Hat OpenShift
- Red Hat Quay (CPK)
- Helm 3
- HashiCorp Vault
- Azure Active Directory
- RHEL CoreOS
- CSI Storage Provisioner

### CI/CD & GitOps (Empfohlen)
- Tekton Pipelines
- Argo CD
- Argo Workflows

### Monitoring & Observability (Empfohlen)
- Grafana
- Prometheus
- Grafana Loki

### Infrastructure Services (Umgebungsabhängig)
- MinIO
- VMware vSphere
- Kasten K10

### Legacy-Systeme (Migration geplant)
- Jenkins (Ablösung durch Tekton)
- VMware vSphere (Cloud-Migration)

## Risikobewertung

### Niedrig
Tools mit stabiler Integration und geringem Ausfallrisiko.

### Mittel
Tools mit Abhängigkeiten oder komplexerer Konfiguration.

### Hoch
Legacy-Systeme oder Tools mit bekannten Problemen.

## Kontakt und Verantwortlichkeiten

- **ICP-Team**: plattform-support@icp-company.de
- **Infrastructure-Team**: plattform-infrastructure@icp-company.de
- **Security-Team**: plattform-security@icp-company.de
- **DevOps-Team**: plattform-devops@icp-company.de
