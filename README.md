# SmartAccess

SmartAccess ist ein Cloud-Service für befristete Zugriffsrechte. Mit Evolveum midPoint sollen Nutzerinnen und Nutzer Berechtigungen beantragen können. Nach der Genehmigung werden die Berechtigungen zugewiesen und nach Ablauf automatisch entzogen.

## Projektstand

SmartAccess wird schrittweise als cloudbasierter IAM-Service auf Basis von Evolveum midPoint und Keycloak entwickelt.

### Abgeschlossene Phasen

- **Phase 1: Repository und Zusammenarbeit:** GitHub-Repository, Teamorganisation und Pull-Request-Workflow eingerichtet.
- **Phase 2: midPoint lokal mit Docker Compose:** midPoint 4.11.0 und PostgreSQL 16 erfolgreich eingerichtet und getestet.
- **Phase 3: Eigenes Multi-Stage-Image:** Optimiertes Keycloak-Image (26.8.2) erstellt und mit einer separaten PostgreSQL-Datenbank in Docker Compose integriert. Build, Anmeldung, Health/Metrics und Persistenz wurden getestet. Die Implementierung wurde über [PR #9](https://github.com/HarrisTane/smartaccess/pull/9) integriert.

### Nächste Phase

**Phase 4 IAM-Datenmodell und Rollen**

Geplant ist die Konfiguration des fachlichen IAM-Modells in midPoint mit drei geschützten Bereichen (IT, Finanzen, Personal), Benutzerrollen sowie Haupt- und Stellvertreter-Genehmigern.

### Noch nicht umgesetzt

- Zugriffsanträge und Genehmigungsworkflows
- Automatische Provisionierung und Entziehung von Keycloak-Berechtigungen
- OpenStack-Infrastruktur und Terraform-Bereitstellung
- Kubernetes-Deployment mit Helm
- CI/CD-Pipeline mit zwei Umgebungen
- Prometheus/Grafana-Monitoring

Die vollständige lokale Installations-, Build- und Testanleitung befindet sich in [app/README.md](app/README.md).

## Repository-Struktur

| Ordner   | Zweck                                                 |
| -------- | ----------------------------------------------------- |
| `app/`   | midPoint-Konfiguration, Dockerfile und Docker Compose |
| `docs/`  | Architektur, Entscheidungen und Betriebshinweise      |
| `infra/` | Terraform und cloud-init für OpenStack                |
| `k8s/`   | Kubernetes-Konfiguration und Helm-Dateien             |


## Lokale Entwicklungsumgebung

SmartAccess wird aktuell lokal mit Docker Compose betrieben.

Die Entwicklungsumgebung besteht aus:

- **Evolveum midPoint 4.11.0:** IAM- und IGA-Anwendung für die spätere Verwaltung von Zugriffsrechten.
- **Keycloak 26.8.2:** Identity Provider und geplantes Zielsystem für die Berechtigungsprovisionierung.
- **PostgreSQL 16:** Separate Datenbanken für midPoint und Keycloak.
- **Docker Compose:** Verwaltung und gemeinsamer Betrieb der Container.

### Lokale Zugriffsadressen

| Dienst | URL |
|---|---|
| midPoint | http://localhost:8080/midpoint/ |
| Keycloak | http://localhost:8081/admin/ |
| Keycloak Health/Metrics | http://localhost:9000 |

Die Dienste sind ausschließlich für den lokalen Entwicklungsbetrieb konfiguriert. Die Provisionierung zwischen midPoint und Keycloak ist noch nicht implementiert.

**Installation, Build, Start, Tests und Fehlerbehebung:**
[Lokale Entwicklungsanleitung](app/README.md)

**Teamorganisation und Zuständigkeiten:**
[Zusammenarbeit im SmartAccess-Team](docs/team.md)



## plan

| Phase | Implementierungsphase               | Koordination | Umsetzung | Review   |
| ----- | ----------------------------------- | ------------ | --------- | -------- |
| 01    | Repository und Zusammenarbeit       | A            | A         | B oder C |
| 02    | midPoint lokal mit Docker Compose   | A            | B         | C        |
| 03    | Eigenes Multi-Stage-Image           | C            | A         | B        |
| 04    | IAM-Datenmodell und Rollen          | A            | B         | C        |
| 05    | Zugriffsanträge und Genehmigungen   | B            | C         | A        |
| 06    | Keycloak-Provisionierung und Ablauf | C            | A         | B        |
| 07    | Terraform und cloud-init            | A            | B         | C        |
| 08    | RKE2-Kubernetes-Cluster             | B            | C         | A        |
| 09    | Helm und Cloud-Deployment           | C            | A         | B        |
| 10    | CI/CD mit GitHub Actions            | A            | B         | C        |
| 11    | Prometheus und Grafana              | B            | C         | A        |
| 12    | Sicherheit und Betriebstests        | C            | A         | B        |
| 13    | End-to-End-Abnahme                  | A            | B         | C        |

## Geplante Cloud-Services-Nachweise

- **Containerisierung** mit Multi-Stage-Dockerfile und Docker Compose
- **Cloud-Infrastruktur** mit OpenStack und cloud-init
- **Infrastructure as Code** mit Terraform, Variablen, Outputs und Remote State
- **Orchestrierung** mit Kubernetes, Helm und Ingress
- **CI/CD** mit GitHub Actions für Build, Tests, Image-Veröffentlichung und Deployment nach `dev` und `staging`
- **Monitoring** mit Prometheus, Grafana und zwei eigenen Dashboards
