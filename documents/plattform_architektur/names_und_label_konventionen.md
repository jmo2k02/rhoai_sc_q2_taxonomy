# ICP Namenskonventionen und Standards

## Überblick

Diese Seite definiert die verbindlichen Namenskonventionen für die Interne Container-Plattform (ICP). Die Einhaltung dieser Standards gewährleistet Konsistenz, Automatisierung und bessere Nachvollziehbarkeit.

## 1. Kubernetes Namespace-Konventionen

### 1.1 Standard-Namensschema
```
<appteam-id>-<umgebung>-<optional-suffix>
```

### 1.2 Umgebungs-Suffixe
| Umgebung | Suffix | Beschreibung | Beispiel |
|----------|--------|--------------|----------|
| Entwicklung | `dev` | Entwicklungsumgebung | `myapp-dev` |
| Qualitätssicherung | `qs` | Test- und QS-Umgebung | `myapp-qs` |
| Integration | `int` | Integrationsumgebung | `myapp-int` |
| Produktion | `prod` | Produktionsumgebung | `myapp-prod` |

### 1.3 Namespace-Beispiele
```bash
# Gültige Namespace-Namen
ecommerce-dev
ecommerce-qs  
ecommerce-prod
finance-api-dev
finance-api-prod
crm-backend-int
```

### 1.4 Namespace-Beschränkungen
- Maximale Länge: 63 Zeichen
- Nur Kleinbuchstaben, Zahlen und Bindestriche
- Muss mit Buchstabe oder Zahl beginnen und enden
- **VERBOTEN**: `default`, `kube-*`, `openshift-*`, `icp-*`

## 2. Container Image-Konventionen

### 2.1 Quay Registry Schema
```
cpk.quay.asg.net/<app-team-name>-<umgebung>/<repo>:<tag>
```

### 2.2 Image-Tag-Konventionen

#### Entwicklungsumgebung
```bash
# Erlaubt - automatische Updates
cpk.quay.asg.net/myapp-dev/backend:latest
cpk.quay.asg.net/myapp-dev/frontend:main
cpk.quay.asg.net/myapp-dev/api:develop
```

#### QS/Integration/Produktion (MUSS-Anforderung)
```bash
# SHA-Digest (bevorzugt ab QSI)
cpk.quay.asg.net/myapp-prod/backend@sha256:a1b2c3d4e5f6...

# Semantische Versionierung (mindestens ab QSI)
cpk.quay.asg.net/myapp-prod/backend:1.2.3
cpk.quay.asg.net/myapp-qs/frontend:2.1.0-rc1

# VERBOTEN in Produktion
cpk.quay.asg.net/myapp-prod/backend:latest  # ❌
cpk.quay.asg.net/myapp-prod/api:main        # ❌
```

### 2.3 Repository-Namenskonventionen
```bash
# Mikroservice-Pattern
<service-name>
<service-name>-<component>

# Beispiele
user-service
payment-api
frontend-web
backend-worker
database-migrations
```

## 3. Applikationsteam-IDs

### 3.1 Team-ID Format
```
<business-unit>-<product-name>
```

### 3.2 Genehmigte Team-IDs (Beispiele)
| Team-ID | Beschreibung | Verantwortlicher |
|---------|--------------|------------------|
| `ecom-shop` | E-Commerce Shop Platform | Team E-Commerce |
| `fin-payment` | Payment Processing | Team Finance |
| `crm-sales` | Sales CRM System | Team Sales |
| `hr-portal` | Human Resources Portal | Team HR |
| `ops-monitoring` | Operations Monitoring | Team DevOps |

### 3.3 Team-ID Registrierung
Neue Team-IDs müssen beim ICP-Team beantragt werden:
- E-Mail an: `plattform-support@icp-company.de`
- Betreff: `Neue Team-ID Beantragung: <gewünschte-id>`
- Inhalt: Business Case und Verantwortliche Person

## 4. Helm Chart-Konventionen

### 4.1 Chart-Namen
```yaml
# Chart.yaml
name: <service-name>
version: <semantic-version>
appVersion: <application-version>

# Beispiel
name: user-service
version: 1.2.3
appVersion: 2.1.0
```

### 4.2 Helm Release-Namen
```bash
<chart-name>-<environment>

# Beispiele
user-service-dev
payment-api-prod
frontend-web-qs
```

### 4.3 Chart Repository Schema
```
oci://cpk.quay.asg.net/<app-team-name>-<umgebung>/helm-charts/<chart-name>
```

## 5. Kubernetes-Ressourcen Konventionen

### 5.1 Labels (MUSS-Anforderungen)
```yaml
metadata:
  labels:
    app.kubernetes.io/name: <app-name>
    app.kubernetes.io/instance: <release-name>
    app.kubernetes.io/version: <app-version>
    app.kubernetes.io/component: <component>
    app.kubernetes.io/part-of: <application>
    app.kubernetes.io/managed-by: helm
    icp.asg.net/team: <team-id>
    icp.asg.net/environment: <env>
```

### 5.2 Service-Namen
```yaml
# Pattern: <app-name>-<component>-<service-type>
metadata:
  name: user-service-api-svc
  name: payment-service-web-svc
  name: frontend-app-ingress-svc
```

### 5.3 ConfigMap/Secret-Namen
```yaml
# Pattern: <app-name>-<purpose>-<type>
metadata:
  name: user-service-app-config
  name: payment-service-db-secret
  name: frontend-app-tls-cert
```

### 5.4 Persistent Volume Claims
```yaml
# Pattern: <app-name>-<component>-<purpose>-pvc
metadata:
  name: user-service-db-data-pvc
  name: payment-service-logs-pvc
  name: frontend-app-assets-pvc
```

## 6. Secret-Management Konventionen

### 6.1 HashiCorp Vault Pfade
```bash
# Pattern: /icp/<team-id>/<environment>/<service>/<secret-type>
/icp/ecom-shop/prod/user-service/database
/icp/fin-payment/dev/api-service/external-keys
/icp/crm-sales/qs/backend/oauth-secrets
```

### 6.2 Kubernetes Secret-Namen
```yaml
# Pattern: <app-name>-<purpose>-vault-secret
metadata:
  name: user-service-db-vault-secret
  name: payment-api-jwt-vault-secret
```

## 7. Netzwerk-Konventionen

### 7.1 Service Ports
| Port Range | Zweck | Beispiel |
|------------|-------|----------|
| 8080-8089 | HTTP Services | 8080, 8081 |
| 8443-8449 | HTTPS Services | 8443, 8444 |
| 9090-9099 | Metrics/Health | 9090 (metrics), 9091 (health) |
| 5432 | PostgreSQL | Standard Port |
| 3306 | MySQL | Standard Port |
| 6379 | Redis | Standard Port |

### 7.2 Ingress-Hostnames
```bash
# Pattern: <service>-<environment>.apps.icp.asg.net
user-service-dev.apps.icp.asg.net
payment-api-prod.apps.icp.asg.net
frontend-qs.apps.icp.asg.net
```

## 8. Monitoring-Labels

### 8.1 Prometheus-Labels
```yaml
metadata:
  labels:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9090"
    prometheus.io/path: "/metrics"
```

### 8.2 Logging-Labels
```yaml
metadata:
  labels:
    logging.coreos.com/collect: "true"
    icp.asg.net/log-level: "info"
    icp.asg.net/log-format: "json"
```

## 9. Compliance und Validierung

### 9.1 Automatische Validierung
Das ICP-Team stellt Admission Controller bereit, die folgende Konventionen durchsetzen:
- Namespace-Namensschema
- Pflicht-Labels
- Image-Tag-Restrictions (ab QSI)
- Resource-Quotas

### 9.2 Policy Violations
Verstöße führen zu:
- **Warnung**: Bei Entwicklungsumgebungen
- **Rejection**: Bei QS/Produktion ohne korrekte Tags
- **Audit Log**: Alle Verstöße werden protokolliert

## 10. Ausnahmen und Genehmigungen

### 10.1 Ausnahmegenehmigung
Ausnahmen von diesen Konventionen erfordern:
1. Schriftlichen Antrag an `plattform-support@icp-company.de`
2. Technische Begründung
3. Genehmigung durch ICP-Team Lead
4. Dokumentation in Confluence

### 10.2 Legacy-Migration
Bestehende Systeme haben eine Übergangsfrist bis **März 2024** zur Anpassung an diese Konventionen.

## Kontakt

Bei Fragen zu Namenskonventionen wenden Sie sich an:
- **ICP-Team**: `plattform-support@icp-company.de`
- **Confluence**: [ICP Namenskonventionen - FAQ](https://confluence.internal.example.com/icp-naming-faq)
