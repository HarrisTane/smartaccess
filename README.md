# SmartAccess

SmartAccess ist ein Cloud-Service für befristete Zugriffsrechte. Mit Evolveum midPoint sollen Nutzerinnen und Nutzer Berechtigungen beantragen können. Nach der Genehmigung werden die Berechtigungen zugewiesen und nach Ablauf automatisch entzogen.

## Projektstand

SmartAccess wird schrittweise als cloudbasierter IAM-Service auf Basis von Evolveum midPoint und Keycloak entwickelt.

### Abgeschlossene Phasen

- **Phase 1: Repository und Zusammenarbeit:** GitHub-Repository, Teamorganisation und Pull-Request-Workflow eingerichtet.
- **Phase 2: midPoint lokal mit Docker Compose:** midPoint 4.11.0 und PostgreSQL 16 erfolgreich eingerichtet und getestet.
- **Phase 3: Eigenes Multi-Stage-Image:** Optimiertes Keycloak-Image (26.8.2) erstellt und mit einer separaten PostgreSQL-Datenbank in Docker Compose integriert. Build, Anmeldung, Health/Metrics und Persistenz wurden getestet. Die Implementierung wurde über [PR #9](https://github.com/HarrisTane/smartaccess/pull/9) integriert.

### Nächste Phase

**Phase 4: CI-Grundlagen mit GitHub Actions**

Wir bauen eine automatisierte CI-Pipeline für SmartAccess auf.
Bei Pull Requests sollen die Docker-Compose-Konfiguration geprüft,
das Keycloak-Image gebaut und automatisierte Tests ausgeführt werden.

Zusätzlich planen wir Sicherheitsprüfungen und die Veröffentlichung
geprüfter Container-Images in der GitHub Container Registry (GHCR).

Die vollständige CD-Integration mit Deployments nach `dev` und
`staging` folgt in Phase 11 nach Kubernetes und Helm.

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



## Implementierungs-Roadmap

| Phase | Implementierungsphase | Koordination | Umsetzung | Review |
| --- | --- | --- | --- | --- |
| 01 | Repository und Zusammenarbeit | A | A | B oder C |
| 02 | midPoint lokal mit Docker Compose | A | B | C |
| 03 | Eigenes Multi-Stage-Image | C | A | B |
| 04 | CI-Grundlagen mit GitHub Actions | A | C | B |
| 05 | IAM-Datenmodell und Rollen | B | A | C |
| 06 | Zugriffsanträge und Genehmigungen | C | B | A |
| 07 | Keycloak-Provisionierung und Ablauf | A | B | C |
| 08 | Terraform und cloud-init | C | A | B |
| 09 | RKE2-Kubernetes-Cluster | B | C | A |
| 10 | Helm und Cloud-Deployment | C | A | B |
| 11 | CI/CD vervollständigen (dev und staging) | A | B | C |
| 12 | Prometheus und Grafana | B | C | A |
| 13 | Sicherheit und Betriebstests | C | A | B |
| 14 | End-to-End-Abnahme | A | B | C |
| 15 | Dokumentation und Abgabe | B | C | A |

## CI-Architektur – Phase 4

**Status:** Architektur beschlossen, technische Umsetzung ausstehend.
**Arbeitsbranch:** `feat/phase-4-CI-foundation`
**GitHub-Issue:** [#11](https://github.com/HarrisTane/smartaccess/issues/11)

### Workflow-Trigger

- Push auf `feat/phase-4-CI-foundation`: Build und Tests.
- Pull Request nach `main`: Automatische Qualitätsprüfung.
- Push auf `main`: Prüfungen und anschließend Image-Veröffentlichung.
- `workflow_dispatch`: Manueller Start für Testzwecke.

Eine Veröffentlichung ist ausschließlich bei einem Push auf `main`
nach erfolgreichen Prüfungen erlaubt.

### CI-Jobs und Abhängigkeiten

1. **Validate:** Docker-Compose-Konfiguration prüfen.
2. **Build:** Keycloak-Multi-Stage-Image erstellen.
3. **Test:** Container starten und Funktionstests ausführen.
4. **Security:** Container-Image auf Schwachstellen prüfen.
5. **Publish:** Geprüftes Image in GHCR veröffentlichen.

Validate → Build → Test und Security parallel → Publish.

Die Jobs Test und Security verwenden dasselbe Image aus dem
Build-Job. Dafür wird das Image als GitHub-Actions-Artefakt
übertragen. Auch Publish verwendet dieses geprüfte Image.

### Automatisierte Tests

- Compose-Konfiguration mit temporären CI-Testwerten validieren.
- Keycloak-Image erfolgreich bauen.
- PostgreSQL-Healthchecks beider Datenbanken überprüfen.
- Keycloak `/health/live` und `/health/ready`: HTTP 200.
- Keycloak `/metrics`: HTTP 200 und Metrikdaten.
- HTTP-Erreichbarkeit von midPoint überprüfen.
- Fehlgeschlagene Pflichtprüfungen müssen den Workflow stoppen.
- Wartezeiten und Wiederholungsversuche begrenzen.

Für CI werden ausschließlich isolierte Testzugangsdaten verwendet.
Die CI-Testumgebung benötigt folgende Pflichtvariablen:

- `POSTGRES_PASSWORD`
- `MIDPOINT_ADMIN_PASSWORD`
- `KEYCLOAK_DB_PASSWORD`
- `KEYCLOAK_ADMIN_PASSWORD`

Diese Werte werden ausschließlich für den isolierten Testlauf
bereitgestellt und dürfen nicht in Images oder Logs gelangen.
Lokale `.env`-Dateien und echte Passwörter werden nicht übertragen.

### Sicherheits- und Veröffentlichungsregeln

- HIGH- und CRITICAL-Schwachstellen blockieren grundsätzlich
  die Image-Veröffentlichung.
- Ausnahmen benötigen eine dokumentierte Risikoanalyse
  und ausdrückliche Freigabe.
- Scannerfehler gelten nicht als bestandene Sicherheitsprüfung.
- GHCR-Schreibrechte erhält ausschließlich der Publish-Job.
- Veröffentlichte Images erhalten nachvollziehbare Commit-SHA-Tags.

### Abgrenzung

Phase 4 umfasst CI, automatisierte Prüfungen und Image-Veröffentlichung.
Das automatische Deployment nach `dev` und `staging` folgt in Phase 11.

## Geplante Cloud-Services-Nachweise

- **Containerisierung** mit Multi-Stage-Dockerfile und Docker Compose
- **Cloud-Infrastruktur** mit OpenStack und cloud-init
- **Infrastructure as Code** mit Terraform, Variablen, Outputs und Remote State
- **Orchestrierung** mit Kubernetes, Helm und Ingress
- **CI/CD** mit GitHub Actions für Build, Tests, Image-Veröffentlichung und Deployment nach `dev` und `staging`
- **Monitoring** mit Prometheus, Grafana und zwei eigenen Dashboards
