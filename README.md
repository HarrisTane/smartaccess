# SmartAccess

SmartAccess ist ein Cloud-Service für befristete Zugriffsrechte. Mit Evolveum midPoint sollen Nutzerinnen und Nutzer Berechtigungen beantragen können. Nach der Genehmigung werden die Berechtigungen zugewiesen und nach Ablauf automatisch entzogen.

## Projektstand

Wir richten in Phase 1 das Repository und die Zusammenarbeit im Dreierteam ein. In Phase 2 folgt der lokale Start von midPoint mit PostgreSQL über Docker Compose.

## Repository-Struktur

| Ordner | Zweck |
| --- | --- |
| `app/` | midPoint-Konfiguration, Dockerfile und Docker Compose |
| `docs/` | Architektur, Entscheidungen und Betriebshinweise |
| `infra/` | Terraform und cloud-init für OpenStack |
| `k8s/` | Kubernetes-Konfiguration und Helm-Dateien |

## Geplante Cloud-Services-Nachweise

- **Containerisierung** mit Multi-Stage-Dockerfile und Docker Compose
- **Cloud-Infrastruktur** mit OpenStack und cloud-init
- **Infrastructure as Code** mit Terraform, Variablen, Outputs und Remote State
- **Orchestrierung** mit Kubernetes, Helm und Ingress
- **CI/CD** mit GitHub Actions für Build, Tests, Image-Veröffentlichung und Deployment nach `dev` und `staging`
- **Monitoring** mit Prometheus, Grafana und zwei eigenen Dashboards
