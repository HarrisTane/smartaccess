# SmartAccess: midPoint lokal starten
 
SmartAccess nutzt Evolveum midPoint mit PostgreSQL. Diese Anleitung startet beides auf deinem Rechner (Windows oder Mac) mit Docker Compose und zeigt, wie du prüfst, dass Daten einen Neustart der Container überstehen.
 
## 1. Voraussetzungen
 
- Git ist installiert.
- Docker Desktop läuft. Prüfen:
```
docker version
```
  Die Ausgabe muss `Client` und `Server` zeigen.
- Port 8080 ist frei. Sonst schlägt der Start mit einem Port-Fehler fehl (siehe Abschnitt 8).
- Genug Arbeitsspeicher für Docker. `compose.yaml` erlaubt midPoint bis zu 2 GB Java-Speicher.
## 2. Repository holen
 
Terminal: PowerShell (Windows) oder Terminal (Mac), in dem Ordner, in dem du Projekte ablegst.
 
```
git clone https://github.com/HarrisTane/smartaccess.git
cd smartaccess/app
```
 
Alle weiteren Befehle laufen im Ordner `app`. Dort liegen `compose.yaml` und `.env.example`.
 
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
 
Öffne `.env` in VS Code (`code .env`) oder einem Editor deiner Wahl. Trage hinter jedes `=` ein eigenes Passwort ein, ohne Leerzeichen und ohne Anführungszeichen. Die beiden Passwörter müssen verschieden sein:
 
| Variable | Wofür |
| --- | --- |
| `POSTGRES_PASSWORD` | Passwort der Datenbank. midPoint nutzt es im Hintergrund, du tippst es nie ein. |
| `MIDPOINT_ADMIN_PASSWORD` | Passwort für den Weblogin als `administrator`. |
 
Hinweise:
- Das Admin-Passwort muss die Passwort-Richtlinie von midPoint erfüllen, sonst wird der Benutzer `administrator` nicht angelegt. Nimm mindestens 12 Zeichen mit Groß- und Kleinbuchstaben, Ziffern und einem Sonderzeichen.
- Verwende keine Passwörter, die du anderswo nutzt. Schreibe sie nicht in Issues, Chats oder Commits.
## 4. Starten
 
```
docker compose up -d
```
 
Beim ersten Mal lädt Docker die Images herunter und legt die Datenbank an. Das kann einige Minuten dauern.
 
Es starten vier Dienste nacheinander:
 
1. `midpoint-db`: die Datenbank.
2. `db-check`: prüft, ob die Datenbank erreichbar ist und ob sie schon Tabellen enthält. Bei einem Fehler (zum Beispiel falsches Passwort) bricht der Start hier ab.
3. `midpoint-init`: bereitet die Konfiguration vor. Er legt das Datenbankschema nur an, wenn `db-check` eine leere Datenbank gemeldet hat, und prüft danach das Repository.
4. `midpoint`: die Anwendung. Sie startet erst, wenn `midpoint-init` erfolgreich war.
## 5. Prüfen, ob alles läuft
 
Warte nach dem Start ein bis zwei Minuten und prüfe den Status:
 
```
docker compose ps
```
 
Erwartet:
- `midpoint-db` ist `healthy`.
- `midpoint` ist `Up` und nach kurzer Zeit `healthy`. Der Port ist `127.0.0.1:8080`, also nur von deinem Rechner aus erreichbar.
- `db-check` und `midpoint-init` tauchen nicht als laufend auf. Sie beenden sich nach getaner Arbeit. Das ist richtig.
Ob die Datenbank neu angelegt oder weiterverwendet wurde, siehst du im Log:
 
| System | Befehl |
| --- | --- |
| Windows (PowerShell) | `docker compose logs db-check midpoint-init \| Select-String "Zustand\|leer\|erfolgreich"` |
| Mac (Terminal) | `docker compose logs db-check midpoint-init \| grep -E "Zustand\|leer\|erfolgreich"` |
 
`empty` bedeutet: Die Datenbank war leer, das Schema wurde angelegt. `filled` bedeutet: Die Datenbank war schon da, es wurde nichts neu angelegt.
 
Fortschritt von midPoint im Log ansehen (Strg+C beendet nur die Anzeige):
 
```
docker compose logs -f midpoint
```
 
## 6. Weblogin
 
1. Öffne http://localhost:8080/midpoint/ im Browser.
2. Benutzername: `administrator`
3. Passwort: der Wert von `MIDPOINT_ADMIN_PASSWORD` aus deiner `.env`.
Erwartet: Du siehst die Startseite von midPoint.
 
## 7. Persistenztest
 
Ziel: Ein Objekt, das du anlegst, ist nach dem Neuerstellen der Container noch da.
 
1. Lege einen Testbenutzer an. In der deutschen Oberfläche: Menü **Benutzer**, dann **Neuer Benutzer**, dann **Person** wählen. Trage im Feld **Name** `persistenz-test` ein und klicke oben auf **Save**. Erwartet: Eine grüne Meldung erscheint, und der Benutzer steht unter **Alle Benutzer** in der Liste. In einer englischen Oberfläche heißen die Menüpunkte anders (Users, New user).
2. Container entfernen, aber die Volumes behalten. Dieser Befehl hat **kein** `-v`:
```
docker compose down
```
3. Neu starten:
```
docker compose up -d
```
4. Warte, bis `docker compose ps` bei `midpoint` wieder `healthy` zeigt, und melde dich erneut an.
5. Prüfe unter **Benutzer**, **Alle Benutzer**, ob `persistenz-test` noch existiert.
Erwartet: Der Benutzer ist noch da. Die Daten liegen in den benannten Volumes `midpoint_db_data` und `midpoint_home`, nicht in den Containern. Im Log aus Abschnitt 5 steht beim Neustart `filled`.
 
## 8. Wenn etwas nicht klappt
 
| Problem | Ursache und Lösung |
| --- | --- |
| „Passwort fehlt“ oder „Admin-Passwort fehlt“ beim Start | In `.env` fehlt `POSTGRES_PASSWORD` oder `MIDPOINT_ADMIN_PASSWORD`. Datei prüfen, speichern, `docker compose up -d` erneut ausführen. |
| Port 8080 ist belegt | Ein anderes Programm oder ein alter Container nutzt den Port. Mit `docker ps` siehst du laufende Container. Das andere Programm beenden. |
| `db-check` endet mit Fehler (`didn't complete successfully: exit 2`) | Die Datenbank hat die Verbindung abgelehnt. Mit `docker compose logs db-check` siehst du den Grund. Steht dort `password authentication failed`, passt `POSTGRES_PASSWORD` in `.env` nicht zum Passwort im vorhandenen Datenbank-Volume. Eine neue Angabe in `.env` ändert das gespeicherte Datenbankpasswort nicht. Setze `.env` auf den alten Wert zurück oder setze alles zurück (Abschnitt 9). `midpoint-init` und `midpoint` starten in diesem Fall absichtlich nicht. |
| `midpoint-init` endet mit Fehler | `docker compose logs midpoint-init` lesen. Meldung dabei nicht raten, sondern im Team nachfragen. |
| `midpoint` bleibt länger als etwa fünf Minuten `unhealthy` | Log lesen: `docker compose logs --tail 50 midpoint`. Meldung im Team nachfragen. |
| Login mit `administrator` schlägt fehl | Das Admin-Passwort wird laut midPoint-Dokumentation nur beim ersten Start einer leeren Installation gesetzt. Eine spätere Änderung in `.env` wirkt nicht. Siehe Abschnitt 9. |
| Warnung „Found orphan containers“ | Harmlos. Im Ordner `app` laufen noch Container aus einem früheren Projekt mit demselben Projektnamen. |
| Meldung „Network app_default Resource is still in use“ beim `down` | Harmlos. Andere Container nutzen das Netzwerk noch. |
 
## 9. Alles zurücksetzen
 
Nur wenn du lokal neu anfangen willst, zum Beispiel nach einem geänderten Passwort in `.env`. **Das löscht alle lokalen midPoint-Daten**, auch deine Testbenutzer:
 
```
docker compose down -v
```
 
Danach `.env` bei Bedarf anpassen und wie in Abschnitt 4 neu starten. Beim nächsten Start meldet `db-check` wieder `empty`, und das Schema wird neu angelegt.
 
## 10. Aufräumen
 
Stoppen und Daten behalten:
 
```
docker compose down
```
 
Die Datei `.env` bleibt lokal und wird nicht committet.
 
