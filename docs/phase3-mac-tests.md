# Phase 3 – Unabhängiger Mac-Test

## Testumgebung

- Datum: 20.11.2026
- Tester: @newspringh
- Getesteter Implementierungscommit: dc5290e3d6a4a9660ec712c843115f7e2b55fc69
- Betriebssystem: macOS 27.2.1
- Host-Architektur: arm64
- Docker Client/Server: 29.9.3
- Docker Compose: v5.7.0
- Compose-Projekt: smartaccess-phase3-test
- Keycloak: 26.8.2
- PostgreSQL: 16-alpine
- midPoint: 4.11.0

Getestet wurde der vorhandene Checkout mit neu angelegten,
getrennten Projekt-Volumes. Die späteren Änderungen betreffen
die Dokumentation.

## Image-Nachweis

- Image: smartaccess-keycloak:local
- Image-ID: sha256:0407ef4f7bdd5652a116c5fc7cb159843f30949a801299cccee39a560f7dafd0
- Betriebssystem/Architektur: linux/arm64
- Größe: 755639944 Bytes
- Konfigurierter Benutzer: 1000
- Gemeldete Build-Dauer: 74,8 Sekunden

Die Größe ist ein gemessener Ausgangswert. Eine Größenersparnis
gegenüber einem Vergleichsimage wurde nicht untersucht.

## Durchführung und Ergebnisse

Die Compose-Befehle wurden im Ordner app mit
-p smartaccess-phase3-test ausgeführt.

| Prüfung | Durchführung | Tatsächliches Ergebnis |
| --- | --- | --- |
| Konfiguration | docker compose -p smartaccess-phase3-test config --quiet | Ohne Fehler |
| Image-Build | docker compose -p smartaccess-phase3-test build keycloak | Image erfolgreich gebaut |
| Erststart | docker compose -p smartaccess-phase3-test up -d | Neue Projekt-Volumes angelegt; Stack gestartet |
| Containerstatus | docker compose -p smartaccess-phase3-test ps -a | Beide Datenbanken und midPoint healthy; Keycloak Up; db-check und midpoint-init Exited (0) |
| Keycloak Live | HTTP-Abfrage von /health/live auf localhost:9000 | HTTP 200; JSON-Status UP |
| Keycloak Ready | HTTP-Abfrage von /health/ready auf localhost:9000 | HTTP 200; JSON-Status UP einschließlich Datenbankprüfung |
| Metrics | HTTP-Abfrage von /metrics auf localhost:9000 | HTTP 200 und Metrikdaten |
| Keycloak-Login | Anmeldung an localhost:8081/admin/ | Erfolgreich |
| Keycloak-Persistenz | Realm phase3-persistence-test angelegt; Container mit down/up neu erstellt | Realm weiterhin vorhanden |
| midPoint-Login | Anmeldung an localhost:8080/midpoint/ | Erfolgreich |
| midPoint-Persistenz | Benutzer phase3-midpoint-test angelegt; Container mit down/up neu erstellt | Benutzer weiterhin vorhanden |
| Nicht-Root-Benutzer | docker compose -p smartaccess-phase3-test exec -T keycloak id | uid=1000(keycloak) gid=0(root); Benutzer-UID ist nicht 0 |

Bei den Persistenztests wurde kein -v verwendet.
Die HTTP-Statuscodes aller drei Keycloak-Endpunkte wurden
nach dem Neuerstellen der Container erneut mit 200 bestätigt.

## Build und Laufzeit

Das geprüfte Dockerfile verwendet zwei benannte Stufen,
builder und runtime, auf Basis derselben Keycloak-Version.
kc.sh build bereitet PostgreSQL-, Health- und Metrics-Unterstützung
vor. Die Runtime übernimmt das Ergebnis und verwendet
start --optimized.

Der Build-Kontext ist app/keycloak. Die dortige .dockerignore
lässt nur Dockerfile und .dockerignore zu. Die lokale .env
liegt außerhalb dieses Build-Kontexts.

## Grenzen und offene Punkte

- Ein zusätzlicher Test aus einem separat neu geklonten Checkout wurde nicht durchgeführt.
- Die Migration vorhandener Phase-2-Daten wurde nicht getestet. Die alten Volumes wurden nicht gelöscht.
- Der alte Phase-2-Benutzer persistenz-test wurde nicht in die neue Testumgebung übernommen.
- Windows und AMD64 wurden in diesem Test nicht geprüft.
- SSO und Provisionierung zwischen midPoint und Keycloak sind noch nicht umgesetzt.
- Registry-Veröffentlichung, Kubernetes und produktives HTTPS gehören zu späteren Phasen.
- Das abschließende Review durch @Sr-Hr20 steht noch aus.