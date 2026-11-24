# Zusammenarbeit im SmartAccess-Team

- **Person A** – `@HarrisTane`
- **Person B** – `@Sr-Hr20`
- **Person C** – `@newspringh`

Wir arbeiten die Implementierungs-Roadmap in der vereinbarten Reihenfolge ab. Beim Start jeder Phase halten wir im zugehörigen GitHub-Issue fest, wer koordiniert, wer eine Teilaufgabe übernimmt und wer den Pull Request prüft.

| Rolle | Aufgabe |
| --- | --- |
| Koordination | Klärt das Ziel der Phase, pflegt das Issue, übernimmt einen Teil der Umsetzung und führt die Beiträge zusammen. |
| Umsetzung | Übernimmt eine konkrete Teilaufgabe und dokumentiert, wie sie geprüft wurde. |
| Review | Prüft den Pull Request anhand des Issues und stimmt zu oder bittet um Änderungen. |

Wir wechseln die Rollen über die Phasen hinweg, damit alle drei praktisch an verschiedenen Teilen des Projekts arbeiten und Reviews übernehmen.

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