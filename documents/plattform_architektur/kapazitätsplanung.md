# OpenShift Kapazitätsplanung

## Übersicht

Diese Dokumentation beschreibt die strategische Kapazitätsplanung unserer on-premises RedHat OpenShift Container Plattform und dokumentiert die Entwicklung der Ressourcenanforderungen über die letzten Jahre. Sie dient als Grundlage für Budgetplanung, Infrastrukturerweiterungen und die langfristige Ausrichtung der Plattform-Kapazitäten.

---

## Zonenbeschreibung

Unsere OpenShift-Plattform ist in fünf geografisch und funktional getrennte Zonen strukturiert, die jeweils unterschiedliche Anforderungen und Wachstumsmuster aufweisen.

**Zone München (MUC)** fungiert als unser Hauptstandort und beherbergt den Großteil unserer kritischen Produktionsworkloads. Diese Zone umfasst sowohl traditionelle Unternehmensanwendungen als auch moderne Cloud-native Microservices-Architekturen. Der Schwerpunkt liegt auf hochverfügbaren Services mit stringenten SLA-Anforderungen. Aufgrund der zentralen Rolle als Haupt-Datacenter verzeichnet München kontinuierlich das stärkste Wachstum bei allen Ressourcenkategorien. Besonders bemerkenswert ist die Zunahme bei GPU-Ressourcen, die durch den verstärkten Einsatz von Machine Learning und AI-Workloads getrieben wird.

**Zone Frankfurt (FRA)** dient primär als Disaster Recovery Standort und Secondary Site für geschäftskritische Anwendungen. Diese Zone hat sich in den letzten Jahren von einer reinen Backup-Infrastruktur zu einem aktiven Produktionsstandort entwickelt, der sowohl DR-Szenarien als auch Load Balancing zwischen den Standorten unterstützt. Das Wachstum in Frankfurt spiegelt unsere Multi-Site-Strategie wider und zeigt eine konstante Steigerung der Kapazitätsanforderungen, insbesondere bei Storage für Datenreplikation und RAM für In-Memory-Caching-Lösungen.

**Zone Hamburg (HAM)** fokussiert sich auf Development und Testing Workloads sowie auf weniger kritische Produktionsanwendungen. Diese Zone charakterisiert sich durch hohe Volatilität in der Ressourcennutzung, da Entwicklungs- und Testumgebungen häufig auf- und abgebaut werden. Das Wachstum ist geprägt von einem steigenden Bedarf an vCPU-Ressourcen für parallele Build-Prozesse und Container-Orchestrierung. Storage-Anforderungen wachsen durch die Notwendigkeit, verschiedene Versionsstände von Anwendungen und Testdaten vorzuhalten.

**Zone Berlin (BER)** wurde 2023 als neue Zone etabliert und konzentriert sich auf innovative Technologien und Forschungsprojekte. Hier werden experimentelle Workloads, Proof-of-Concepts und neue Technologie-Stacks evaluiert. Die Zone zeigt das dynamischste Wachstum bei GPU-Ressourcen, da hier Machine Learning, Künstliche Intelligenz und High-Performance Computing Anwendungen entwickelt und getestet werden. Die Ressourcenanforderungen sind geprägt von unvorhersehbaren Spitzen und der Notwendigkeit, flexible Kapazitäten für verschiedene Forschungsprojekte bereitzustellen.

**Zone Düsseldorf (DUS)** fungiert als regionale Hub für kundenspezifische Anwendungen und lokale Services. Diese Zone unterstützt primär Anwendungen mit geringen Latenzanforderungen und regionalen Compliance-Anforderungen. Das Wachstum ist moderat aber stetig, getrieben durch die Expansion regionaler Services und die Verlagerung von Legacy-Anwendungen in die Container-Umgebung. Besonders bemerkenswert ist der Anstieg bei Storage-Anforderungen, da hier historische Daten und Archive migriert werden.

---

## Kapazitätsbedarfe und Entwicklung

### Gemeldete Kapazitätsbedarfe 2022-2025

| Jahr | Zone | vCPU Kerne | vCPU Mehrbedarf | GPU Kerne | GPU Mehrbedarf | RAM (GB) | RAM Mehrbedarf | Storage (TB) | Storage Mehrbedarf |
|------|------|------------|-----------------|-----------|----------------|----------|----------------|--------------|--------------------|
| **2022** | MUC | 2.840 | - | 32 | - | 11.360 | - | 156 | - |
| | FRA | 1.420 | - | 16 | - | 5.680 | - | 89 | - |
| | HAM | 1.136 | - | 8 | - | 4.544 | - | 67 | - |
| | DUS | 568 | - | 4 | - | 2.272 | - | 34 | - |
| | BER | - | - | - | - | - | - | - | - |
| | **Gesamt** | **5.964** | **-** | **60** | **-** | **23.856** | **-** | **346** | **-** |
| **2023** | MUC | 3.692 | +852 (+30%) | 56 | +24 (+75%) | 14.768 | +3.408 (+30%) | 201 | +45 (+29%) |
| | FRA | 1.846 | +426 (+30%) | 28 | +12 (+75%) | 7.384 | +1.704 (+30%) | 115 | +26 (+29%) |
| | HAM | 1.477 | +341 (+30%) | 14 | +6 (+75%) | 5.908 | +1.364 (+30%) | 87 | +20 (+30%) |
| | DUS | 738 | +170 (+30%) | 7 | +3 (+75%) | 2.952 | +680 (+30%) | 44 | +10 (+29%) |
| | BER | 454 | +454 (neu) | 8 | +8 (neu) | 1.816 | +1.816 (neu) | 28 | +28 (neu) |
| | **Gesamt** | **8.207** | **+2.243 (+38%)** | **113** | **+53 (+88%)** | **32.828** | **+8.972 (+38%)** | **475** | **+129 (+37%)** |
| **2024** | MUC | 4.430 | +738 (+20%) | 78 | +22 (+39%) | 17.720 | +2.952 (+20%) | 251 | +50 (+25%) |
| | FRA | 2.215 | +369 (+20%) | 39 | +11 (+39%) | 8.860 | +1.476 (+20%) | 144 | +29 (+25%) |
| | HAM | 1.772 | +295 (+20%) | 20 | +6 (+43%) | 7.088 | +1.180 (+20%) | 109 | +22 (+25%) |
| | DUS | 886 | +148 (+20%) | 10 | +3 (+43%) | 3.544 | +592 (+20%) | 55 | +11 (+25%) |
| | BER | 681 | +227 (+50%) | 16 | +8 (+100%) | 2.724 | +908 (+50%) | 42 | +14 (+50%) |
| | **Gesamt** | **9.984** | **+1.777 (+22%)** | **163** | **+50 (+44%)** | **39.936** | **+7.108 (+22%)** | **601** | **+126 (+27%)** |
| **2025** | MUC | 5.316 | +886 (+20%) | 101 | +23 (+29%) | 21.264 | +3.544 (+20%) | 314 | +63 (+25%) |
| | FRA | 2.658 | +443 (+20%) | 51 | +12 (+31%) | 10.632 | +1.772 (+20%) | 180 | +36 (+25%) |
| | HAM | 2.126 | +354 (+20%) | 26 | +6 (+30%) | 8.504 | +1.416 (+20%) | 136 | +27 (+25%) |
| | DUS | 1.063 | +177 (+20%) | 13 | +3 (+30%) | 4.252 | +708 (+20%) | 69 | +14 (+25%) |
| | BER | 1.022 | +341 (+50%) | 26 | +10 (+63%) | 4.088 | +1.364 (+50%) | 63 | +21 (+50%) |
| | **Gesamt** | **12.185** | **+2.201 (+22%)** | **217** | **+54 (+33%)** | **48.740** | **+8.804 (+22%)** | **762** | **+161 (+27%)** |

### Entwicklungstrends nach Ressourcentyp

**vCPU-Entwicklung:** Die vCPU-Anforderungen zeigen ein konstantes Wachstum über alle Zonen hinweg, wobei München als Hauptstandort erwartungsgemäß die höchsten absoluten Zuwächse verzeichnet. Das Gesamtwachstum von 5.964 Kernen in 2022 auf prognostizierte 12.185 Kerne in 2025 entspricht einer Verdopplung der CPU-Kapazitäten. Bemerkenswert ist die Stabilisierung der Wachstumsrate bei etwa 20-22% pro Jahr nach dem initialen Wachstumsschub von 2022 auf 2023.

**GPU-Entwicklung:** GPU-Ressourcen zeigen das dynamischste Wachstum aller Ressourcenkategorien, mit einer Steigerung von 60 Kernen in 2022 auf prognostizierte 217 Kerne in 2025. Dieses Wachstum wird primär durch die Einführung von Machine Learning Workloads, AI-Anwendungen und High-Performance Computing getrieben. Berlin als Innovationsstandort zeigt hier die stärksten relativen Zuwächse, während München aufgrund seiner Größe die höchsten absoluten Zahlen aufweist.

**RAM-Entwicklung:** Die Speicheranforderungen wachsen proportional zu den CPU-Anforderungen, was auf eine ausgewogene Workload-Verteilung hinweist. Von 23.856 GB in 2022 auf prognostizierte 48.740 GB in 2025 verdoppelt sich auch hier die Kapazität. Die gleichmäßige Wachstumsrate von etwa 20-22% zeigt eine vorhersagbare Entwicklung, die durch den Trend zu speicherintensiven Anwendungen und In-Memory-Datenbanken getrieben wird.

**Storage-Entwicklung:** Storage-Anforderungen wachsen mit etwa 25-27% pro Jahr am stärksten, was auf die zunehmende Datenintensität moderner Anwendungen und die Notwendigkeit für Backup- und Archivierungsstrategien hinweist. Von 346 TB in 2022 auf prognostizierte 762 TB in 2025 mehr als verdoppelt sich die benötigte Speicherkapazität. Frankfurt zeigt hier besonders starkes Wachstum aufgrund seiner Rolle als DR-Standort.

---

## Kapazitätsplanung und Strategische Ausrichtung

**Mittelfristige Prognose (2026-2027):** Basierend auf den aktuellen Trends erwarten wir eine Fortsetzung des Wachstums bei moderaten Raten. Für 2026 prognostizieren wir einen Gesamtbedarf von etwa 15.000 vCPU-Kernen, 280 GPU-Kernen, 60.000 GB RAM und 950 TB Storage. Die Wachstumsraten werden sich voraussichtlich bei etwa 18-20% stabilisieren, da die initiale Migrationswelle von Legacy-Systemen abebbt.

**Investitionspriorisierung:** GPU-Kapazitäten erfordern die höchste Aufmerksamkeit aufgrund der starken Nachfrage und hohen Beschaffungskosten. Eine strategische Vorabinvestition in GPU-Ressourcen wird empfohlen, um Engpässe zu vermeiden. Storage-Wachstum erfordert kontinuierliche Investitionen in Hochleistungsspeicher und Backup-Infrastruktur.

**Zonale Entwicklungsstrategie:** Berlin als Innovationsstandort wird überproportional wachsen und erfordert flexible Kapazitätsplanungen. München bleibt der Haupttreiber für absolute Kapazitätserweiterungen. Frankfurt entwickelt sich zu einem vollwertigen Produktionsstandort und benötigt entsprechende Infrastrukturinvestitionen.

**Kostenoptimierung:** Durch die Implementierung von Cluster-Autoscaling und ressourcenbasierter Abrechnung können Effizienzgewinne von 15-20% realisiert werden. Die Einführung von Spot-Instances für nicht-kritische Workloads kann weitere Kosteneinsparungen ermöglichen.

---

## Empfehlungen und Nächste Schritte

Die Kapazitätsplanung sollte quartalsweise überprüft und angepasst werden, um auf sich ändernde Geschäftsanforderungen reagieren zu können. Die Implementierung von Monitoring-Tools für Ressourcennutzung und Predictive Analytics wird empfohlen, um die Planungsgenauigkeit zu verbessern.

Eine strategische Partnerschaft mit Hardware-Anbietern für GPU-Kapazitäten sollte etabliert werden, um Lieferengpässe zu vermeiden. Die Evaluierung von Hybrid-Cloud-Szenarien für Spitzenlasten kann die Kapazitätsflexibilität erhöhen, ohne die Grundinvestition zu erhöhen.

---

*Letzte Aktualisierung: Juni 2025 | Version: 1.0 | Erstellt von: Platform Engineering Team*