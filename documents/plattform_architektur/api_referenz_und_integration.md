# ICP API-Referenz und Integration

## Überblick

Diese Seite dokumentiert die spezifischen APIs und Integrationen der Internen Container-Plattform (ICP), einschließlich Custom Resources, Webhooks und automatisierter Workflows.

## 1. ICP Custom Resources

### 1.1 TeamOnboarding CRD

Die `TeamOnboarding` Custom Resource automatisiert das Onboarding neuer Applikationsteams. Sie erstellt automatisch Namespaces, Quotas, RBAC-Bindings und initialisiert Monitoring.

**Wichtige Felder:**
- `teamId`: Eindeutige Team-ID (Format: `<business-unit>-<product>`)
- `businessUnit`: Zuordnung zur Organisationseinheit
- `environments`: Liste der benötigten Umgebungen
- `resourceQuotas`: Umgebungsspezifische Ressourcenverteilung
- `securityProfile`: Standard-, Enhanced- oder Custom-Sicherheitsprofil

**Automatische Aktionen nach Erstellung:**
1. Namespace-Erstellung mit ICP-Labels
2. ResourceQuota- und LimitRange-Anwendung
3. Azure AD Group-Mapping
4. Quay-Organisation-Erstellung
5. ServiceMonitor-Templates
6. NetworkPolicy-Basis-Setup

### 1.2 ImageImportRequest CRD

Diese CRD steuert den kontrollierten Import externer Container-Images in die ICP-Registry.

**Workflow-Phasen:**
- `Pending`: Anfrage eingegangen
- `Scanning`: Sicherheitsscan läuft (Twistlock/Trivy)
- `Approved/Rejected`: Basierend auf CVE-Score
- `Importing`: Transfer zu cpk.quay.asg.net
- `Ready`: Image verfügbar

**Automatische Sicherheitsprüfungen:**
- CVE-Scan mit konfigurierbarem Schwellenwert
- Base-Image-Validierung gegen erlaubte Registry-Liste  
- Lizenz-Compliance-Check
- Malware-Scanning

### 1.3 DisasterRecoveryPolicy CRD

Definiert RTO/RPO-Anforderungen und automatisiert DR-Konfigurationen.

**Service-Tier-Mapping:**
- **Platinum**: RTO <15min, RPO <5min, Synchrone Replikation
- **Gold**: RTO <1h, RPO <15min, Kasten-Backup alle 15min
- **Silver**: RTO <4h, RPO <1h, Stündliche Backups
- **Bronze**: RTO <24h, RPO <4h, Tägliche Backups

## 2. ICP Admission Webhooks

### 2.1 Resource-Validation Webhook

**Endpoint:** `https://icp-webhook-service.icp-system.svc:443/validate-resources`

**Validierungsregeln:**

| Kategorie | Regel | Aktion bei Verletzung |
|-----------|-------|----------------------|
| **Pflicht-Labels** | app.kubernetes.io/name, icp.asg.net/team | Rejection |
| **Resource-Limits** | CPU Request ≥ 50m, Memory Request ≥ 64Mi | Rejection |
| **Security Context** | runAsNonRoot=true, privileged=false | Rejection |
| **Image-Registry** | Nur cpk.quay.asg.net und registry.redhat.io | Rejection |
| **Production-Tags** | Keine :latest/:main in QS/Prod | Rejection |

### 2.2 Namespace-Enforcement Webhook

**Automatische Namespace-Konfiguration:**
- Injection von Standard-NetworkPolicies
- LimitRange-Anwendung basierend auf Umgebung
- PodSecurityStandards-Enforcement
- ServiceAccount-Auto-Creation

### 2.3 Cost-Optimization Webhook

**Mutating Webhook für Ressourcen-Optimierung:**
- Automatisches Hinzufügen von CPU/Memory-Requests wenn fehlend
- HPA-Empfehlungen für stateless Workloads
- PVC-Größen-Validierung gegen Storage-Quotas
- Ineffiziente Resource-Ratios Warnung

## 3. ICP REST API

### 3.1 Team-Management API

**Base URL:** `https://api.icp.asg.net/v1`

#### Team-Registrierung
```http
POST /teams
Content-Type: application/json
Authorization: Bearer <DAS-Token>

{
  "teamId": "ecom-checkout",
  "businessUnit": "ecommerce", 
  "contact": "checkout-team@icp-company.de",
  "environments": ["dev", "qs", "prod"],
  "estimatedWorkloads": 5,
  "expectedTraffic": "medium"
}
```

**Response:** Team-Onboarding-Status und generierte Namespace-Liste

#### Quota-Anfrage
```http
POST /teams/{teamId}/quota-request
Content-Type: application/json

{
  "environment": "prod",
  "justification": "Black Friday traffic spike",
  "requestedQuota": {
    "cpu": "16",
    "memory": "32Gi", 
    "storage": "1Ti"
  },
  "duration": "30 days",
  "businessCase": "EUR 2M additional revenue expected"
}
```

### 3.2 Image-Management API

#### Image-Import-Anfrage
```http
POST /images/import
Content-Type: application/json

{
  "sourceImage": "docker.io/nginx:1.21.0",
  "targetNamespace": "ecom-frontend-prod",
  "securityScanRequired": true,
  "maxCvssScore": 7.0,
  "approvalWorkflow": "automatic"
}
```

#### Image-Scan-Status
```http
GET /images/import/{requestId}/status

Response:
{
  "phase": "Ready",
  "vulnerabilities": {
    "critical": 0,
    "high": 2, 
    "medium": 8,
    "low": 23
  },
  "importedImage": "cpk.quay.asg.net/ecom-frontend-prod/nginx@sha256:abc123...",
  "scanReport": "https://api.icp.asg.net/v1/images/import/req-123/report"
}
```

### 3.3 Monitoring API

#### Health-Check Aggregation
```http
GET /health/{namespace}

Response:
{
  "overall": "healthy",
  "services": {
    "frontend": {"status": "healthy", "replicas": "3/3"},
    "api": {"status": "degraded", "replicas": "2/3"},
    "database": {"status": "healthy", "replicas": "2/2"}
  },
  "resourceUtilization": {
    "cpu": "68%",
    "memory": "72%",
    "storage": "45%"
  },
  "alerts": ["api-pod-crashlooping"]
}
```

## 4. Automation Workflows

### 4.1 GitOps-Integration

**ArgoCD Application-Set für neue Teams:**
Der ICP-Controller erstellt automatisch ArgoCD ApplicationSets basierend auf TeamOnboarding CRDs.

**Naming Convention:** `{teamId}-{environment}-appset`

**Auto-Generated Application Pattern:**
- Repository: `https://git.company.com/{teamId}/k8s-manifests`
- Path: `environments/{environment}`
- Target Namespace: `{teamId}-{environment}`
- Sync Policy: Automated mit Pruning

### 4.2 Backup-Automation

**Kasten K10 Policy-Generator:**
Basierend auf DisasterRecoveryPolicy CRDs werden automatisch Kasten-Policies erstellt.

**Policy-Naming:** `{namespace}-{service-tier}-backup`

**Automatische Konfiguration:**
- Backup-Frequenz basierend auf Service-Tier
- Cross-Region-Export für Platinum/Gold
- Retention-Policies nach Compliance-Anforderungen
- Pre/Post-Hooks für Datenbank-Konsistenz

### 4.3 Security-Compliance Automation

**Continuous Compliance Scanning:**
- Täglich: Image-Vulnerability-Scans aller laufenden Container
- Wöchentlich: NetworkPolicy-Compliance-Check
- Monatlich: RBAC-Permission-Audit
- Quartalsweise: Secret-Rotation-Validation

**Automatische Remediation:**
- Image-Updates bei kritischen CVEs
- NetworkPolicy-Gaps automatisch schließen
- Expired Secret-Alerts
- Non-compliant Pod-Termination

## 5. Integration Points

### 5.1 Azure AD Integration

**LDAP-Query-Syntax für Team-Mapping:**
```
(&(objectClass=group)(cn=ICP-*)(description=*{teamId}*))
```

**Group-Naming-Convention:**
- `ICP-Admins-{TeamId}-{Environment}` → namespace-admin Role
- `ICP-Users-{TeamId}-{Environment}` → namespace-edit Role  
- `ICP-Viewers-{TeamId}-{Environment}` → namespace-view Role

**Automated Group-Sync Frequency:** Alle 4 Stunden

### 5.2 Jira Integration

**Webhook-Endpoints:**
- `/webhooks/jira/quota-requests` - Automatische Ticket-Erstellung bei Quota-Überschreitung
- `/webhooks/jira/incidents` - Bi-direktionale Incident-Synchronisation
- `/webhooks/jira/onboarding` - Team-Onboarding-Workflow-Integration

**Custom Fields in Jira:**
- `ICP-Namespace`: Betroffener Namespace
- `ICP-Team-ID`: Zuständiges Team
- `ICP-Environment`: Umgebung (dev/qs/prod)
- `ICP-Service-Tier`: Priorität basierend auf Service-Tier

### 5.3 Confluence Integration

**Automated Documentation Generation:**
- Team-spezifische Runbook-Seiten
- Auto-generierte API-Dokumentation
- Compliance-Report-Publishing
- Architecture-Decision-Records (ADR) Templates

**Content-Sync via REST API:**
- Wöchentlich: Namespace-Ressource-Overview
- Bei Changes: Onboarding-Status-Updates
- Monatlich: Cost-Reports und Utilization-Metrics

## 6. Metrics und Observability

### 6.1 Custom Metrics

**ICP-spezifische Prometheus Metrics:**

| Metric | Beschreibung | Labels |
|--------|-------------|---------|
| `icp_team_onboarding_duration_seconds` | Zeit bis Team-Setup abgeschlossen | team_id, environment |
| `icp_image_import_total` | Anzahl Image-Import-Requests | source_registry, status |
| `icp_quota_utilization_ratio` | Quota-Auslastung pro Namespace | namespace, resource_type |
| `icp_backup_success_total` | Erfolgreiche Backups | namespace, service_tier |
| `icp_cost_per_namespace_euros` | Monatliche Kosten pro Namespace | namespace, team_id |

### 6.2 SLI/SLO Definitionen

**Service Level Indicators:**

| SLI | Definition | Target |
|-----|------------|---------|
| **Team Onboarding Time** | Zeit von Anfrage bis produktiver Namespace | <4 Stunden |
| **Image Import Success Rate** | Erfolgreiche Image-Imports/Gesamt | >99% |
| **Backup Success Rate** | Erfolgreiche Backups/Geplant | >99.9% |
| **API Response Time** | P95 Response-Zeit ICP APIs | <500ms |
| **Webhook Availability** | Admission Webhook Verfügbarkeit | >99.95% |

## 7. Troubleshooting APIs

### 7.1 Debug-Information API

**Namespace Health Check:**
```http
GET /debug/{namespace}/health
```

Aggregiert automatisch:
- Pod-Status und Recent Events
- Resource-Utilization vs. Quotas  
- NetworkPolicy-Violations
- Recent Failed Deployments
- PVC-Mount-Issues

### 7.2 Performance-Analyse API

**Resource-Recommendation Engine:**
```http
GET /optimize/{namespace}/recommendations

Response:
{
  "rightsizing": [
    {
      "workload": "frontend-deployment",
      "currentResources": {"cpu": "1000m", "memory": "2Gi"},
      "recommendedResources": {"cpu": "500m", "memory": "1Gi"},
      "estimatedSavings": "EUR 45/month"
    }
  ],
  "hpaOpportunities": ["api-service", "worker-deployment"],
  "unusedResources": [
    "pvc/old-logs-storage (not mounted for 30 days)"
  ]
}
```

## 8. Error Handling und Rate Limiting

### 8.1 API Error Codes

| HTTP Code | ICP Error Code | Bedeutung |
|-----------|---------------|-----------|
| 400 | ICP-1001 | Invalid Team-ID format |
| 403 | ICP-2001 | Insufficient RBAC permissions |
| 409 | ICP-3001 | Team-ID already exists |
| 422 | ICP-4001 | Resource quota exceeds cluster capacity |
| 429 | ICP-5001 | Rate limit exceeded |
| 503 | ICP-6001 | Backend service unavailable |

### 8.2 Rate Limiting

**Per-User Limits (basierend auf DAS-ID):**
- 100 requests/hour für Standard-Users
- 1000 requests/hour für Admin-Users
- 10 concurrent connections max

**Per-Team Limits:**
- 5 Image-Import-Requests/hour
- 2 Quota-Change-Requests/day
- 1 Team-Onboarding/week