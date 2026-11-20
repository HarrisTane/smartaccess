# SmartAccess

SmartAccess ist ein Cloud-Service für befristete Zugriffsrechte. Mit Evolveum midPoint sollen Nutzerinnen und Nutzer Berechtigungen beantragen können. Nach der Genehmigung werden die Berechtigungen zugewiesen und nach Ablauf automatisch entzogen.

## Projektstand

Phase 2 ist abgeschlossen. midPoint und PostgreSQL können lokal mit Docker Compose betrieben werden.

In Phase 3 ergänzen wir ein eigenes, optimiertes Keycloak-Image mit Multi-Stage-Build und einer separaten PostgreSQL-Datenbank. Build, Weblogin, Health/Metrics und Datenpersistenz wurden auf einem Mac mit Apple Silicon getestet. Dokumentation und abschließendes Review werden noch vervollständigt.

Die Verbindung zwischen midPoint und Keycloak für die Provisionierung folgt in Phase 6 und ist noch nicht umgesetzt.

## Repository-Struktur

| Ordner | Zweck |
| --- | --- |
| `app/` | midPoint-Konfiguration, Dockerfile und Docker Compose |
| `docs/` | Architektur, Entscheidungen und Betriebshinweise |
| `infra/` | Terraform und cloud-init für OpenStack |
| `k8s/` | Kubernetes-Konfiguration und Helm-Dateien |

## plan

| Phase | Implementierungsphase | Koordination | Umsetzung | Review |
| --- | --- | --- | --- | --- |
| 01 | Repository und Zusammenarbeit | A | A | B oder C |
| 02 | midPoint lokal mit Docker Compose | A | B | C |
| 03 | Eigenes Multi-Stage-Image | C | A | B |
| 04 | IAM-Datenmodell und Rollen | A | B | C |
| 05 | Zugriffsanträge und Genehmigungen | B | C | A |
| 06 | Keycloak-Provisionierung und Ablauf | C | A | B |
| 07 | Terraform und cloud-init | A | B | C |
| 08 | RKE2-Kubernetes-Cluster | B | C | A |
| 09 | Helm und Cloud-Deployment | C | A | B |
| 10 | CI/CD mit GitHub Actions | A | B | C |
| 11 | Prometheus und Grafana | B | C | A |
| 12 | Sicherheit und Betriebstests | C | A | B |
| 13 | End-to-End-Abnahme | A | B | C |

## Geplante Cloud-Services-Nachweise

- **Containerisierung** mit Multi-Stage-Dockerfile und Docker Compose
- **Cloud-Infrastruktur** mit OpenStack und cloud-init
- **Infrastructure as Code** mit Terraform, Variablen, Outputs und Remote State
- **Orchestrierung** mit Kubernetes, Helm und Ingress
- **CI/CD** mit GitHub Actions für Build, Tests, Image-Veröffentlichung und Deployment nach `dev` und `staging`
- **Monitoring** mit Prometheus, Grafana und zwei eigenen Dashboards
