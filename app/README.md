# SmartAccess: midPoint und Keycloak lokal starten

Diese Anleitung beschreibt den lokalen Betrieb von midPoint und einem eigenen, optimierten Keycloak-Image mit Docker Compose. Beide Dienste verwenden jeweils eine separate PostgreSQL-Datenbank.

Das Keycloak-Image wird mit einem Multi-Stage-Dockerfile gebaut. Die Anleitung umfasst Build, Start, Weblogin, Health-/Metrics-Prüfungen und Persistenztests.

Die Verbindung zwischen midPoint und Keycloak für die Provisionierung folgt in Phase 6 und ist hier noch nicht eingerichtet.

## 1. Voraussetzungen

- Git ist installiert.
- Docker Desktop läuft. Prüfen:
```
docker version
```
  Die Ausgabe muss `Client` und `Server` zeigen.
- Die lokalen Ports 8080 (midPoint), 8081 (Keycloak) und 9000 (Keycloak-Management) sind frei. Die veröffentlichten Ports sind ausschließlich an 127.0.0.1 gebunden.
- Genug Arbeitsspeicher für Docker. `compose.yaml` erlaubt midPoint bis zu 2 GB Java-Speicher.

## 2. Repository holen

Das Repository klonen und in den Projektordner wechseln:

```bash
git clone https://github.com/HarrisTane/smartaccess.git
cd smartaccess
```

Solange Phase 3 noch nicht in `main` integriert ist, den Implementierungsbranch auswählen:

```bash
git switch feat/phase-3-keycloak-image
```

Nach der Integration der vollständigen Phase 3 in `main` ist dieser zusätzliche Branchwechsel nicht mehr erforderlich.

Anschließend in den Anwendungsordner wechseln:

```bash
cd app
```

Alle weiteren Befehle laufen im Ordner `app`. Dort liegen `compose.yaml` und `.env.example`. Das Dockerfile und die .dockerignore liegen unter `keycloak/`.

## 3. Lokale Zugangsdaten einrichten

Die Datei `.env` enthält deine Passwörter. Sie bleibt lokal und darf nie ins Git. Prüfe, dass Git sie ignoriert:

```
git check-ignore .env
```

Die Ausgabe muss `.env` lauten. Bei leerer Ausgabe nicht weitermachen und im Team melden.

Datei anlegen:

| System | Befehl |
| --- | --- |
| Windows (PowerShell) | `copy .env.example .env` |
| Mac (Terminal) | `cp .env.example .env` |

Öffne `.env` in VS Code (`code .env`) oder einem Editor deiner Wahl. Trage hinter jedes `=` ein eigenes Passwort ein, ohne Leerzeichen und ohne Anführungszeichen. Für jede der vier Variablen ein eigenes, unterschiedliches Passwort eintragen:

| Variable | Wofür |
| --- | --- |
| `POSTGRES_PASSWORD` | Passwort der midPoint-Datenbank. |
| `MIDPOINT_ADMIN_PASSWORD` | Weblogin bei midPoint als `administrator`. |
| `KEYCLOAK_DB_PASSWORD` | Passwort der separaten Keycloak-Datenbank. |
| `KEYCLOAK_ADMIN_PASSWORD` | Weblogin bei Keycloak als `admin`. |

Hinweise:
- Das Admin-Passwort muss die Passwort-Richtlinie von midPoint erfüllen, sonst wird der Benutzer `administrator` nicht angelegt. Nimm mindestens 12 Zeichen mit Groß- und Kleinbuchstaben, Ziffern und einem Sonderzeichen.
- Verwende keine Passwörter, die du anderswo nutzt. Schreibe sie nicht in Issues, Chats oder Commits.

## 4. Image bauen und Stack starten

Die folgenden Befehle werden im Ordner `app` ausgeführt. Für einen getrennten lokalen Test verwenden wir durchgehend den Projektnamen `smartaccess-phase3-test`. Dadurch entstehen eigene Netzwerke und Volumes. Andere laufende Dienste dürfen die Ports 8080, 8081 und 9000 nicht belegen.

Zuerst die Konfiguration prüfen, ohne Zugangsdaten auszugeben:

```bash
docker compose -p smartaccess-phase3-test config --quiet
```

Das eigene Keycloak-Image bauen:

```bash
docker compose -p smartaccess-phase3-test build keycloak
```

Das Dockerfile verwendet Keycloak 26.8.2 in zwei benannten Stufen:

- `builder`: Bereitet Keycloak mit `kc.sh build` für PostgreSQL, Health und Metrics vor. Dafür werden keine laufende Datenbank und keine Zugangsdaten benötigt.
- `runtime`: Übernimmt das vorbereitete Ergebnis und startet Keycloak mit `start --optimized`.

Datenbankadresse, Zugangsdaten, Hostname und die lokale HTTP-Konfiguration werden erst beim Start über Compose bereitgestellt. Die `.env` gehört nicht in das Image. Der Build-Kontext ist auf `app/keycloak` begrenzt; dessen `.dockerignore` lässt nur Dockerfile und .dockerignore zu.

Der Multi-Stage-Build verlagert die Vorbereitung in den Image-Build. Eine bestimmte Größenersparnis wird damit nicht behauptet.

Anschließend den Stack starten:

```bash
docker compose -p smartaccess-phase3-test up -d
```

Der Stack enthält sechs Dienste:

- `midpoint-db` → `db-check` → `midpoint-init` → `midpoint`
- `keycloak-db` → `keycloak`

Die Anwendungen warten auf ihre jeweiligen Voraussetzungen. PostgreSQL verwendet das Image `postgres:16-alpine`. Beide Datenbanken haben getrennte benannte Volumes.

HTTP ist hier ausschließlich für den lokalen Test vorgesehen. HTTPS/Ingress und die produktive Secret-Verwaltung folgen in späteren Phasen.

## 5. Status, Health und Metrics prüfen

Nach dem Start kurz warten und alle Container anzeigen:

```bash
docker compose -p smartaccess-phase3-test ps -a
```

Erwartet:

- `midpoint-db` und `keycloak-db` sind `healthy`.
- `db-check` und `midpoint-init` sind mit `Exited (0)` beendet.
- `midpoint` ist nach dem Start `healthy`.
- `keycloak` ist `Up`. Seine Bereitschaft wird über die folgenden HTTP-Abfragen geprüft; `Up` allein reicht dafür nicht aus.

Die folgenden curl-Befehle sind für das Terminal unter macOS/Linux vorgesehen. Unter Windows gegebenenfalls `curl.exe` verwenden.

```bash
curl --fail --silent --show-error --write-out '\nHTTP %{http_code}\n' http://localhost:9000/health/live
curl --fail --silent --show-error --write-out '\nHTTP %{http_code}\n' http://localhost:9000/health/ready
curl --fail --silent --show-error --write-out '\nHTTP %{http_code}\n' http://localhost:9000/metrics
```

Erwartet:

- `/health/live`: HTTP 200 und JSON-Status `UP`.
- `/health/ready`: HTTP 200 und JSON-Status `UP`, einschließlich der Datenbankprüfung.
- `/metrics`: HTTP 200 und Metrikdaten mit Einträgen wie `# HELP` und `# TYPE`.

Keycloak läuft ohne Root-Benutzer. Prüfen:

```bash
docker compose -p smartaccess-phase3-test exec -T keycloak id
```

Im Mac-Test war das Ergebnis `uid=1000(keycloak) gid=0(root)`. Die UID ist nicht 0; die Gruppenzugehörigkeit ist davon zu unterscheiden.

Den erkannten Zustand der midPoint-Datenbank unter macOS/Linux prüfen:

```bash
docker compose -p smartaccess-phase3-test logs db-check midpoint-init | grep -E 'Zustand|vorhanden|ueberspringe|erfolgreich'
```

`empty` kennzeichnet eine leere Datenbank. Bei vorhandenen Tabellen wird `filled` gemeldet und die Schema-Erstellung übersprungen.

## 6. Weblogin

### midPoint

1. http://localhost:8080/midpoint/ öffnen.
2. Benutzername: `administrator`
3. Passwort: Wert von `MIDPOINT_ADMIN_PASSWORD` aus der lokalen `.env`.

Erwartet: Die Anmeldung ist erfolgreich und die midPoint-Startseite erscheint.

### Keycloak

1. http://localhost:8081/admin/ öffnen.
2. Benutzername: `admin`
3. Passwort: Wert von `KEYCLOAK_ADMIN_PASSWORD` aus der lokalen `.env`.

Erwartet: Die Anmeldung an der Keycloak-Admin-Konsole ist erfolgreich.

Die Zugangsdaten werden bei der Erstinitialisierung verwendet. Eine spätere Änderung der `.env` ändert keine bereits gespeicherten Datenbank- oder Administratorpasswörter.

Beide Anwendungen werden in dieser Phase unabhängig geprüft. Eine Anmeldung über einen gemeinsamen Identity Provider sowie die Provisionierung zwischen midPoint und Keycloak sind damit noch nicht nachgewiesen.


## 7. Persistenztest für beide Anwendungen

Ziel: Testobjekte bleiben nach dem Entfernen und Neuerstellen der Container mit denselben Volumes erhalten.

### Testobjekte anlegen

1. In midPoint anmelden und unter **Users → New user** einen Benutzer mit dem Namen `phase3-midpoint-test` anlegen und speichern. In der deutschen Oberfläche heißen die Menüpunkte entsprechend **Benutzer → Neuer Benutzer**.
2. Prüfen, dass der Benutzer unter **Users → All users** erscheint.
3. In der Keycloak-Admin-Konsole über **Manage realms → Create realm** einen Realm namens `phase3-persistence-test` anlegen.
4. Prüfen, dass der Realm in der Realm-Auswahl erscheint.

### Container neu erstellen

Im Ordner `app` ausführen:

```bash
docker compose -p smartaccess-phase3-test down
docker compose -p smartaccess-phase3-test up -d
```

Dabei kein `-v` verwenden. Den Projektnamen beibehalten, damit dieselben Volumes verwendet werden.

Nach dem Start die Bereitschaft wie in Abschnitt 5 prüfen. Anschließend erneut an beiden Anwendungen anmelden.

Erwartet:

- Der Keycloak-Realm `phase3-persistence-test` ist weiterhin vorhanden.
- Der midPoint-Benutzer `phase3-midpoint-test` ist weiterhin vorhanden.
- Beide Weblogins funktionieren.
- `db-check` meldet für midPoint `filled`; das Schema wird nicht erneut angelegt.

Die Daten liegen in den projektbezogenen Volumes für `midpoint_db_data`, `keycloak_db_data` und `midpoint_home`.

### Abgrenzung zum Upgrade aus Phase 2

Der dokumentierte Mac-Test verwendet mit `smartaccess-phase3-test` eine getrennte Neuinstallation und neue Testobjekte. Die Migration vorhandener Phase-2-Daten wurde damit nicht geprüft.

Insbesondere hat sich der logische Name des midPoint-Datenbank-Volumes von `postgres_data` auf `midpoint_db_data` geändert. Ein vorhandenes Phase-2-Volume wird dadurch nicht automatisch übernommen. Alte Volumes nicht ungeprüft löschen.

## 8. Wenn etwas nicht klappt

| Problem | Prüfung und Vorgehen |
| --- | --- |
| Compose meldet ein fehlendes Passwort | Alle vier Variablen in der lokalen `.env` prüfen. Danach `docker compose -p smartaccess-phase3-test config --quiet` ausführen. |
| Port 8080, 8081 oder 9000 ist belegt | Mit `docker ps` prüfen, welcher Dienst den Port verwendet. Nur bekannte, nicht benötigte Dienste stoppen. |
| `db-check` endet mit Exit 2 | Lokal das Log von `db-check` prüfen. Bei `password authentication failed` den ursprünglichen Datenbank-Passwortwert wiederherstellen. Eine Änderung der `.env` ändert kein gespeichertes Datenbankpasswort. |
| Keycloak startet nicht oder ist nicht bereit | Status und lokale Logs von `keycloak` und `keycloak-db` prüfen. Datenbankverbindung, Laufzeitwerte und Build-Ergebnis kontrollieren. |
| Health oder Metrics liefern kein HTTP 200 | Startbereitschaft, Port 9000 und Build-Optionen prüfen. Nach Änderungen am Dockerfile das Image neu bauen und den Container neu erstellen. |
| Login schlägt nach einer Passwortänderung in `.env` fehl | Die Änderung rotiert kein bereits gespeichertes Passwort. Den bisherigen gültigen Wert verwenden oder das Passwort gezielt im jeweiligen System ändern. |
| Warnung zu verwaisten Containern | Prüfen, zu welchem Stack die Container gehören. Keine fremden Container ungeprüft entfernen. |

Logs können vertrauliche Werte enthalten. Nur lokal prüfen und keine ungefilterten Logs oder `.env`-Inhalte veröffentlichen.

## 9. Neuinstallation und vorhandene Daten

Der Test mit dem Projektnamen `smartaccess-phase3-test` verwendet eigene Volumes. Die Trennung der Daten gibt die Host-Ports nicht frei; diese dürfen nur von einem Stack gleichzeitig belegt werden.

Eine Neuinstallation ersetzt keinen Migrationstest. Vorhandene Phase-2-Daten bleiben außerhalb dieses Tests. Ihre Übernahme erfordert eine gesonderte Planung und Prüfung.

Das Löschen von Volumes entfernt die darin gespeicherten Daten. Für die beschriebenen Start-, Stopp- und Persistenztests ist keine Volume-Löschung erforderlich.

## 10. Stoppen und erneut starten

Container stoppen und entfernen, Daten behalten:

```bash
docker compose -p smartaccess-phase3-test down
```

Mit denselben Volumes erneut starten:

```bash
docker compose -p smartaccess-phase3-test up -d
```

Den Projektnamen bei allen Befehlen beibehalten. Die lokale `.env` bleibt außerhalb von Git.