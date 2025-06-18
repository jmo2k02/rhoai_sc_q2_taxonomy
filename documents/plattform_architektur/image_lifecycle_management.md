# OpenShift Container Image Lifecycle Management

## Übersicht

Diese Seite dokumentiert den Image Lifecycle Prozess der ICP. Sie beschreibt die verschiedenen Arten von Container Images, deren Promotion-Pfade durch die Umgebungen und die Lifecycle-Management-Strategien.

---

## 1. Container Image Kategorien

### 1.1 Base Images
**Beschreibung:** Grundlegende Betriebssystem-Images, die als Basis für Anwendungsimages dienen.

**Beispiele:**
- Red Hat Universal Base Images (UBI)
- Alpine Linux
- Custom Enterprise Base Images

**Charakteristika:**

Diese Images werden zentral durch unser Security Team verwaltet und unterliegen strengen Härtungsrichtlinien, um eine minimale Angriffsfläche zu gewährleisten. Regelmäßige Sicherheitsupdates werden systematisch eingespielt, wobei die Compliance-Konformität stets gewährleistet bleibt. Die Base Images dienen als vertrauensvolle Grundlage für alle darauf aufbauenden Container-Schichten.

### 1.2 Runtime Images
**Beschreibung:** Images mit vorinstallierten Laufzeitumgebungen für spezifische Technologien.

**Beispiele:**
- Java Runtime (OpenJDK, IBM JDK)
- Node.js Runtime
- Python Runtime
- .NET Core Runtime

**Charakteristika:**

Diese Images werden durch unser Platform Engineering Team entwickelt und gepflegt, wobei sie stets auf den aktuellen gehärteten Base Images aufbauen. Die Runtime Images enthalten optimierte Konfigurationen für die jeweiligen Laufzeitumgebungen und werden regelmäßig auf Performance und Sicherheit überprüft.

### 1.3 Application Images
**Beschreibung:** Images mit deployten Anwendungen, die auf Runtime Images aufbauen. Sie repräsentieren die eigentlichen Geschäftsanwendungen und werden durch die jeweiligen Entwicklungsteams erstellt.

**Beispiele:**
- Microservices
- Web-Anwendungen
- APIs
- Batch-Jobs

**Charakteristika:**
Diese Images basieren auf den standardisierten Runtime Images und enthalten den spezifischen Anwendungscode samt aller erforderlichen Abhängigkeiten. Sie folgen strengen Naming Conventions und werden durch automatisierte CI/CD-Pipelines erstellt und getestet.

### 1.4 Tool Images
**Beschreibung:** Spezialisierte Images für Build-, Test- und Deployment-Prozesse.

**Beispiele:**
- CI/CD Build Agents
- Testing Frameworks
- Deployment Tools
- Monitoring Agents

**Charakteristika:**
Diese Images werden zentral durch unser DevOps Team verwaltet und enthalten spezifische Tools und Konfigurationen, die für die automatisierten Entwicklungs- und Betriebsprozesse erforderlich sind.

---

## 2. Image Promotion Pipeline

### 2.1 Umgebungslandschaft

```
Entwicklung → Staging → Pre-Production → Production
     ↓            ↓           ↓              ↓
   dev-ns      staging-ns   preprod-ns    prod-ns
```

### 2.2 Promotion-Prozess

#### Phase 1: Development
**Verantwortlichkeit:** Entwicklungsteams

**Prozess:**
1. Entwickler erstellt Dockerfile basierend auf approved Base/Runtime Images
2. Lokaler Build und Tests
3. Push zu Development Registry
4. Automatische Bereitstellung in Development Namespace

**Registry:** `cpk.quay.asg.net/<app-team-name>-dev/<repo>:<tag>`

**Gating Criteria:**
- Successful Build
- Unit Tests bestanden
- Code Quality Gates erfüllt

#### Phase 2: Staging
**Verantwortlichkeit:** QA Teams

**Prozess:**
1. Automatischer Trigger nach erfolgreichem Development Build
2. Image wird zu Staging Registry promoted
3. Deployment in Staging Umgebung
4. Automatisierte Integration Tests

**Registry:** `cpk.quay.asg.net/<app-team-name>-staging/<repo>:<tag>`

**Gating Criteria:**
- Security Scan bestanden (Critical/High Vulnerabilities = 0)
- Integration Tests erfolgreich
- Performance Tests bestanden

#### Phase 3: Pre-Production
**Verantwortlichkeit:** Release Management

**Prozess:**
1. Manuelle Promotion nach Staging Approval
2. Image wird zu Pre-Production Registry promoted
3. Smoke Tests in produktionsähnlicher Umgebung
4. Change Management Approval

**Registry:** `cpk.quay.asg.net/<app-team-name>-preprod/<repo>:<tag>`

**Gating Criteria:**
- Change Approval erteilt
- Smoke Tests bestanden
- Rollback Plan dokumentiert

#### Phase 4: Production
**Verantwortlichkeit:** Operations Team

**Prozess:**
1. Geplante Promotion nach Pre-Production Approval
2. Image wird zu Production Registry promoted
3. Blue-Green oder Canary Deployment
4. Monitoring und Health Checks

**Registry:** `cpk.quay.asg.net/<app-team-name>-prod/<repo>:<tag>`

**Gating Criteria:**
- Maintenance Window genehmigt
- Monitoring Alerts konfiguriert
- Rollback verified

### 2.3 Automatisierung

**CI/CD Pipeline Tools:**
- **Jenkins/Tekton:** Build und Test Automation
- **ArgoCD:** GitOps-basierte Deployments
- **Quay:** Enterprise Registry mit Security Scanning
- **Twistlock/Aqua:** Runtime Security Monitoring

**Promotion Trigger:**
- Development → Staging: Automatisch nach erfolgreichem Build
- Staging → Pre-Production: Manuell nach QA Approval  
- Pre-Production → Production: Geplant nach Change Approval

---

## 3. Image Lifecycle Management

### 3.1 Retention Policies

#### Development Images
- **Retention:** 30 Tage
- **Grund:** Kurze Entwicklungszyklen, häufige Updates
- **Cleanup:** Automatisch durch Cron Job

#### Staging Images  
- **Retention:** 90 Tage
- **Grund:** Längere Testzyklen, Regression Testing
- **Cleanup:** Automatisch mit Benachrichtigung

#### Pre-Production Images
- **Retention:** 180 Tage  
- **Grund:** Rollback-Fähigkeit, Compliance
- **Cleanup:** Manuell nach Review

#### Production Images
- **Retention:** 2 Jahre
- **Grund:** Compliance, Audit, Long-term Support
- **Cleanup:** Geplant nach Lifecycle Ende

### 3.2 Vulnerability Management

#### Scanning Schedule
- **Daily:** Alle Production Images
- **Weekly:** Pre-Production und Staging Images
- **On-Push:** Development Images

#### Remediation Process
1. **Critical Vulnerabilities:** Sofortige Benachrichtigung, 24h Remediation SLA
2. **High Vulnerabilities:** 7 Tage Remediation SLA
3. **Medium Vulnerabilities:** 30 Tage Remediation SLA
4. **Low Vulnerabilities:** Nächster geplanter Update-Zyklus

#### Quarantine Process
Images mit Critical Vulnerabilities werden automatisch:
- Aus Production Registries entfernt
- Mit Quarantine-Tag versehen
- Für Deployment gesperrt

### 3.3 Compliance und Auditing

#### Image Signierung
- Alle Production Images werden mit Cosign signiert
- Private Keys werden in HashiCorp Vault verwaltet
- Signaturen werden bei Deployment verifiziert

#### Audit Logs
- Vollständige Nachverfolgung aller Image Operationen
- Integration mit SIEM System
- Monatliche Compliance Reports

#### Backup und Disaster Recovery
- Tägliche Backups aller Production Images
- Cross-Region Replikation
- RTO: 4 Stunden, RPO: 24 Stunden

---

## 4. Governance und Prozesse

### 4.1 Image Standards

#### Naming Convention
```
[registry]/[organisation]/[repo]:[version]-[build-number]
```

**Beispiel:**
```
cpk.quay.asg.net/ki_poc-staging/inferencer:1.0.0-1
```

#### Required Labels
- `version`: Anwendungsversion
- `build-date`: Build Timestamp
- `git-commit`: Git Commit Hash
- `maintainer`: Verantwortliches Team
- `security-scan`: Letzter Scan Timestamp

### 4.2 Security Hardening

#### Mandatory Requirements
- Non-root User Execution
- Keine Secrets in Image Layers
- Minimale Package Installation
- Security Updates automatisch angewendet

#### Scanning Integration
- Trivy für Vulnerability Scanning  
- OPA Gatekeeper für Policy Enforcement
- Falco für Runtime Monitoring

### 4.3 Monitoring und Alerting

#### Metriken
- Image Pull Rates
- Vulnerability Trends
- Storage Usage
- Build Success Rates

#### Alerts
- Failed Security Scans
- Quota Exceeded
- Deprecated Image Usage
- Compliance Violations

---

## 5. Tools und Integration

### 5.1 Registry Infrastructure
- **Production:** Red Hat Quay Enterprise
- **Development:** Red Hat Quay Enterprise
- **Backup:** AWS ECR (Disaster Recovery)

### 5.2 Automation Stack
- **CI/CD:** Jenkins mit OpenShift Pipeline Plugin
- **GitOps:** ArgoCD für Deployment Automation
- **Security:** Twistlock für Runtime Protection
- **Monitoring:** Prometheus + Grafana

### 5.3 Integration Points
- **LDAP:** User Authentication und Authorization
- **ServiceNow:** Change Management Integration
- **Slack:** Notification und Alerting
- **JIRA:** Issue Tracking und Workflow

---

## 6. Kontakte und Verantwortlichkeiten

| Rolle | Team | Kontakt |
|-------|------|---------|
| Platform Engineering | DevOps Team | plattform-devops@icp-company.de |
| Security | Security Team | plattform-security@icp-company.de |
| Release Management | Release Team | plattform-release@icp-company.de |
| Operations | Operations Team | plattform-ops@icp-company.de |

---

## 7. Weitere Ressourcen

- [OpenShift Image Management Best Practices](internal-link)
- [Security Hardening Guidelines](internal-link)
- [Disaster Recovery Procedures](internal-link)
- [Compliance Documentation](internal-link)

