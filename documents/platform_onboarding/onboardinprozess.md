# OpenShift Container Plattform - Onboarding Prozess

## Überblick

Diese Seite dokumentiert den strukturierten Onboardingprozess für neue Applikationsteams auf der RedHat OpenShift Container Plattform. Der Prozess ist in fünf Hauptphasen unterteilt und folgt dem VABI-Verantwortlichkeitsmodell (Verantwortlich, Ausführend, Beratend, Informiert).

## VABI-Verantwortlichkeitsmatrix

| Aufgabe | Beschreibung | Plattform-Team | Applikationsteam | Externe Abhängigkeit |
|---------|--------------|:--------------:|:----------------:|:--------------------:|
| **ORGANISATION** |
| Verfahrensname | Definition und Abstimmung des offiziellen Verfahrensnamens für das Onboarding-Projekt zwischen allen Beteiligten | V, A | V, A | I |
| Applikationsteam Organisation | Identifikation der Teamstruktur, Definition von Rollen und Benennung der Ansprechpartner für das Projekt | B | V, A | I |
| Confluence | Erstellung der Confluence-Projektseite mit Dokumentationsstruktur und Zugriffsberechtigungen | A, B | V | I |
| Team-Mailbox | Einrichtung einer dedizierten Funktions-Mailbox für das Applikationsteam mit entsprechenden Verteilerlisten | I | V, A | A (IT-Service-Desk) |
| Workloadbeschreibung | Detaillierte technische und fachliche Beschreibung der geplanten Container-Workloads und Anwendungsarchitektur | B | V, A | I |
| Akzeptanzkriterien | Definition messbarer Kriterien für erfolgreiche Plattform-Bereitstellung mit formeller Bestätigung durch Applikationsteam | B | V, A | I |
| Betrieb | Entwicklung des Betriebskonzepts inkl. SLA-Definition, Monitoring-Anforderungen und Eskalationsprozesse | V, A | A, B | I |
| Termine | Koordination aller Projektmeilensteine und Abhängigkeiten mit Terminplanung für Go-Live | V, A | A, B | B |
| Datenschutz | Bewertung datenschutzrechtlicher Anforderungen und Erstellung des Datenschutz-Impact-Assessments | B | V, A | V, A (Datenschutzbeauftragte) |
| **RESSOURCEN** |
| Bedarfsermittlung | Detaillierte Analyse der benötigten Compute-, Memory-, Storage- und Netzwerk-Ressourcen basierend auf Workload-Anforderungen | V, A | A, B | I |
| Applikationslog-Volumina-Prognose | Schätzung der erwarteten Log-Datenmengen für Dimensionierung der Logging-Pipeline und Storage-Anforderungen | B | V, A | I |
| **NETZWERK** |
| Container Plattform Kommunikationsmatrix | Dokumentation aller internen Pod-zu-Pod und Service-zu-Service Kommunikationswege innerhalb der Plattform | V, A | A, B | I |
| Ingress/Egress Traffic Kommunikationsmatrix | Spezifikation aller eingehenden und ausgehenden Netzwerkverbindungen mit Ports, Protokollen und Zielen | A, B | V, A | I |
| Firewall extern | Beantragung und Konfiguration externer Firewall-Regeln für Ingress/Egress Traffic der Container-Workloads | B | A | V, A (Netzwerk-Security-Team) |
| Externe Schnittstellen | Identifikation und Dokumentation aller externen API-Abhängigkeiten, Datenbank-Verbindungen und Service-Integrationen | I | V, A | A, B (Enterprise-Architecture) |
| **TECHNISCHE INFORMATIONEN** |
| Plattform Services | Bereitstellung und Konfiguration der OpenShift-Basisdienste wie Registry, Monitoring, Logging und Service Mesh | V, A | I | I |
| RBAC / AD Integration | Einrichtung der Benutzerauthentifizierung, Gruppenmapping und Integration mit Active Directory-Services | V, A | A, B | A (Identity-Management-Team) |
| Namespaces | Erstellung und Konfiguration der Kubernetes-Namespaces entsprechend der Umgebungsstrategie (dev/test/prod) | V, A | B | I |
| Berechtigungen | Zuweisung spezifischer OpenShift-Rollen und Kubernetes-RBAC-Policies basierend auf Prinzip der minimalen Berechtigung | V, A | A, B | I |
| Quota | Definition und Implementierung von Ressourcen-Quotas pro Namespace zur Gewährleistung fairer Ressourcenverteilung | V, A | A, B | I |
| LimitRange | Konfiguration von Default- und Maximum-Ressourcenlimits für Container zur Vermeidung von Resource-Starvation | V, A | B | I |
| **UMSETZUNG** |
| Meldung an Capacity Management | Weiterleitung der finalen Ressourcenanforderungen an Capacity Management für Hardware-Provisioning | V, A | I | I |
| Bereitstellung der Ressourcen auf Clusterebene | Physische Bereitstellung zusätzlicher Cluster-Nodes und Storage-Kapazitäten entsprechend Sizing | A | I | V, A (Infrastructure-Team) |
| Confluence | Abschluss der technischen Dokumentation und Übergabe der Betriebsdokumentation an das Applikationsteam | V, A | A, B | I |
| Namespace einrichten | Technische Implementierung der Namespace-Konfiguration mit allen definierten Policies und Ressourcen-Limits | V, A | I | I |
| Plattform Services | Finale Aktivierung aller Plattform-Services mit Durchführung der Integrationstests und Übergabe an den Betrieb | V, A | A, B | I |

## VABI-Legende

- **V (Verantwortlich)**: Trägt die Gesamtverantwortung für die Aufgabe und deren Ergebnis
- **A (Ausführend)**: Führt die Aufgabe praktisch durch und arbeitet an der Umsetzung
- **B (Beratend)**: Stellt Expertise zur Verfügung und unterstützt beratend bei der Durchführung
- **I (Informiert)**: Wird über Fortschritt und Ergebnisse informiert, aber nicht aktiv eingebunden

## Externe Abhängigkeiten - Abteilungen

- **IT-Service-Desk**: Bereitstellung von IT-Basisservices wie E-Mail-Systeme
- **Datenschutzbeauftragte**: Bewertung und Freigabe datenschutzrechtlicher Aspekte
- **Netzwerk-Security-Team**: Firewall-Management und Netzwerk-Sicherheitsrichtlinien
- **Enterprise-Architecture**: Architektur-Governance und Schnittstellenstandards
- **Identity-Management-Team**: Active Directory und Benutzer-/Gruppenverwaltung
- **Infrastructure-Team**: Hardware-Bereitstellung und Rechenzentrumsinfrastruktur

## Projektphasen und Meilensteine

1. **Organisationsphase** (Woche 1-2): Projektsetup und Teamorganisation
2. **Planungsphase** (Woche 3-4): Ressourcen- und Netzwerkplanung
3. **Konfigurationsphase** (Woche 5-6): Technische Implementierung
4. **Umsetzungsphase** (Woche 7-8): Deployment und Integrationstests
5. **Go-Live** (Woche 9): Produktivschaltung und Übergabe an den Betrieb

## Eskalationspfade

Bei Problemen oder Verzögerungen erfolgt die Eskalation über:
1. **Technische Eskalation**: Plattform-Team → Infrastructure-Team → IT-Operations-Manager
2. **Fachliche Eskalation**: Applikationsteam → Projekt-Sponsor → Business-Owner
3. **Projekt-Eskalation**: Projekt-Koordination → IT-Portfolio-Management

## Kontakt und Support

- **Plattform-Team**: plattform-support@icp-company.de
- **Projekt-Koordination**: plattform-onboarding@icp-company.de
- **24/7 Support**: plattform-support@icp-company.de