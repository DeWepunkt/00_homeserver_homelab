# Migration vom Raspberry Pi 500 auf den Homeserver

## Projektziel

Die bestehenden Homelab-Dienste sollen vom Raspberry Pi 500 auf den neuen Homeserver migriert werden.

Ziel ist es, die vorhandene Umgebung mit Portainer, Uptime Kuma, PostgreSQL und Adminer auf dem HP ProDesk 800 G5 Mini weiterzuführen.

Dabei sollen bestehende Konfigurationen und Daten möglichst erhalten bleiben.

---

## Ausgangssituation

Das Homelab lief ursprünglich auf einem Raspberry Pi 500.

Auf dem Raspberry Pi 500 waren bereits mehrere Homelab-Dienste als Docker-Container eingerichtet:

* Portainer
* Uptime Kuma
* PostgreSQL
* Adminer

Damit wurde nicht ein vollständiges Betriebssystem migriert, sondern die bestehenden Container-Dienste mit ihren relevanten Daten und Konfigurationen.

Die Dienste liefen im lokalen Heimnetz und waren nicht über Portfreigaben aus dem Internet erreichbar.

Der neue Homeserver wurde zuvor mit Debian Server minimal und RAID 1 eingerichtet. Damit stand eine neue Grundlage für den weiteren Betrieb der Homelab-Dienste bereit.

---

## Migrationsstrategie

Die Migration wurde nicht durch Kopieren des gesamten Systems durchgeführt.

Stattdessen wurden die relevanten Docker-Daten gezielt übertragen:

* Portainer-Daten über Docker-Volume
* Uptime-Kuma-Daten über Docker-Volume
* PostgreSQL-Daten über logischen SQL-Dump
* Adminer ohne eigene persistente Daten

Für PostgreSQL wurde bewusst kein Rohkopieren des Docker-Volumes verwendet.

Der Raspberry Pi 500 nutzt eine ARM-Plattform, der neue Homeserver eine x86-Plattform. Bei Datenbanken ist ein logischer Export und Import in diesem Fall die sauberere und besser nachvollziehbare Lösung.

Dadurch wird nicht die interne Datenstruktur des alten Containers kopiert, sondern der Datenbankinhalt über PostgreSQL selbst exportiert und auf dem neuen System wieder importiert.

---

## Vorbereitung auf dem Raspberry Pi 500

Zunächst wurden die vorhandenen Docker-Volumes und Container auf dem Raspberry Pi 500 geprüft.

Relevante Volumes:

* `portainer_data`
* `uptime-kuma`
* `postgres_data`

Relevante Container:

* `portainer`
* `uptime-kuma`
* `postgres`
* `adminer`

Für die Migration wurde ein Exportverzeichnis erstellt:

```bash
mkdir -p ~/migration-export
```

---

## Container auf dem Raspberry Pi stoppen

Vor dem Export wurden die betroffenen Container gestoppt.

```bash
docker stop adminer uptime-kuma postgres portainer
```

Damit wird verhindert, dass sich Daten während des Exports ändern.

Besonders bei Datenbanken und Konfigurationsdaten ist ein konsistenter Zustand wichtig.

---

## Docker-Volumes exportieren

Die Docker-Volumes von Portainer und Uptime Kuma wurden als Archivdateien exportiert.

Dafür wurde ein temporärer Alpine-Container verwendet, der das jeweilige Volume einbindet und den Inhalt als `.tar.gz`-Archiv erstellt.

### Portainer-Volume exportieren

```bash
docker run --rm \
  -v portainer_data:/volume \
  -v ~/migration-export:/backup \
  alpine \
  tar czf /backup/portainer_data.tar.gz -C /volume .
```

### Uptime-Kuma-Volume exportieren

```bash
docker run --rm \
  -v uptime-kuma:/volume \
  -v ~/migration-export:/backup \
  alpine \
  tar czf /backup/uptime-kuma.tar.gz -C /volume .
```

Die Archivdateien wurden im Verzeichnis `~/migration-export` gespeichert.

---

## PostgreSQL per SQL-Dump exportieren

PostgreSQL wurde nicht über das Docker-Volume migriert, sondern über einen logischen Dump.

Dafür wurde `pg_dumpall` im laufenden PostgreSQL-Container verwendet:

```bash
docker exec -t postgres pg_dumpall -U postgres > ~/migration-export/postgres_dump.sql
```

Der Dump enthält die PostgreSQL-Datenbanken und Rollen des alten Systems.

Das echte Datenbankpasswort wird nicht in der Dokumentation gespeichert.

---

## Übertragung auf den Homeserver

Die Exportdateien wurden vom Raspberry Pi 500 auf den neuen Homeserver übertragen.

Beispiel:

```bash
scp ~/migration-export/portainer_data.tar.gz <benutzername>@192.168.x.x:~/migration-export/
scp ~/migration-export/uptime-kuma.tar.gz <benutzername>@192.168.x.x:~/migration-export/
scp ~/migration-export/postgres_dump.sql <benutzername>@192.168.x.x:~/migration-export/
```

Die IP-Adresse steht dabei für die lokale Adresse des Homeservers im Heimnetz.

---

## Vorbereitung auf dem Homeserver

Auf dem Homeserver wurde ebenfalls ein Verzeichnis für die Migrationsdateien verwendet:

```bash
mkdir -p ~/migration-export
```

Außerdem wurde ein Arbeitsverzeichnis für den Docker-Compose-Stack angelegt:

```bash
mkdir -p ~/docker/homelab
```

Der Compose-Projektordner heißt `homelab`. Der Hostname des Servers lautet `homeserver`.

Damit sind Hostname und Compose-Projektname getrennt:

* `homeserver`: Name des Debian-Systems im Netzwerk
* `homelab`: Name des Docker-Compose-Projekts

---

## Docker-Volumes auf dem Homeserver erstellen

Für die wiederherzustellenden Dienste wurden die benötigten Docker-Volumes erstellt.

```bash
docker volume create portainer_data
docker volume create uptime-kuma
```

Das PostgreSQL-Volume wird später durch Docker Compose automatisch erstellt.

---

## Docker-Volumes wiederherstellen

Die gesicherten Volumes wurden auf dem Homeserver aus den Archivdateien wiederhergestellt.

### Portainer-Volume wiederherstellen

```bash
docker run --rm \
  -v portainer_data:/volume \
  -v ~/migration-export:/backup \
  alpine \
  sh -c "cd /volume && tar xzf /backup/portainer_data.tar.gz"
```

### Uptime-Kuma-Volume wiederherstellen

```bash
docker run --rm \
  -v uptime-kuma:/volume \
  -v ~/migration-export:/backup \
  alpine \
  sh -c "cd /volume && tar xzf /backup/uptime-kuma.tar.gz"
```

Damit stehen die bisherigen Konfigurationen von Portainer und Uptime Kuma auf dem neuen Homeserver wieder zur Verfügung.

---

## Docker Compose auf dem Homeserver

Auf dem Homeserver werden die Dienste über Docker Compose verwaltet.

Die Compose-Datei liegt im Verzeichnis:

```text
~/docker/homelab
```

Die Datei `docker-compose.yml` enthält die Dienste:

* Portainer
* Uptime Kuma
* PostgreSQL
* Adminer

Beispielhafte Compose-Struktur:

```yaml
services:
  portainer:
    image: portainer/portainer-ce:lts
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9443:9443"
      - "8000:8000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - uptime-kuma:/app/data

  postgres:
    image: postgres:16
    container_name: postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  adminer:
    image: adminer:latest
    container_name: adminer
    restart: unless-stopped
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  portainer_data:
    external: true
  uptime-kuma:
    external: true
  postgres_data:
```

---

## Umgang mit Zugangsdaten

Die PostgreSQL-Zugangsdaten werden nicht direkt in der `docker-compose.yml` gespeichert.

Stattdessen wird eine `.env`-Datei im gleichen Verzeichnis verwendet.

Beispielhafte Struktur:

```text
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<starkes-passwort>
POSTGRES_DB=postgres
```

Das echte Passwort wird nicht in der Projektdokumentation gespeichert.

Die `.env`-Datei darf nicht in das Git-Repository aufgenommen werden.

Zusätzlich wurden die Dateirechte eingeschränkt:

```bash
chmod 600 .env
```

Damit kann nur der Besitzer die Datei lesen und ändern.

---

## Docker-Compose-Konfiguration prüfen

Vor dem Start der Dienste wurde die Compose-Konfiguration geprüft.

Dabei wurde bewusst `--quiet` verwendet:

```bash
docker compose config --quiet
```

Dieser Befehl prüft die Konfiguration, ohne die aufgelösten Umgebungsvariablen auszugeben.

Das ist wichtig, weil `docker compose config` ohne `--quiet` die Werte aus der `.env`-Datei anzeigen kann.

---

## Dienste auf dem Homeserver starten

Nach erfolgreicher Prüfung wurden die Dienste gestartet:

```bash
docker compose up -d
```

Der Parameter `-d` startet die Container im Hintergrund.

Anschließend wurde geprüft, ob die Container laufen:

```bash
docker ps
```

Erwartete Container:

* `portainer`
* `uptime-kuma`
* `postgres`
* `adminer`

---

## PostgreSQL-Dump importieren

Nachdem der neue PostgreSQL-Container auf dem Homeserver lief, wurde der SQL-Dump importiert.

```bash
docker exec -i postgres psql -U postgres -d postgres < ~/migration-export/postgres_dump.sql
```

Ein erster Importversuch schlug aufgrund eines unvollständigen Befehls fehl.

Die Ursache war kein Datenbankproblem, sondern ein fehlerhaft eingegebener Docker-Befehl.

Nach Korrektur des Befehls konnte der Dump importiert werden.

---

## Zugriff auf Adminer prüfen

Adminer wurde im Browser über die lokale Adresse des Homeservers geöffnet:

```text
http://192.168.x.x:8080
```

Für die Anmeldung wurden folgende Werte verwendet:

```text
System: PostgreSQL
Server: postgres
User: postgres
Password: <Passwort aus .env>
Database: postgres
```

Der Servername `postgres` funktioniert, weil Adminer und PostgreSQL im gleichen Docker-Compose-Projekt laufen und sich intern über den Servicenamen erreichen können.

---

## Zugriff auf PostgreSQL von Arbeitsgeräten

Für lokale Entwicklungs- und Administrationswerkzeuge kann PostgreSQL im Heimnetz über den Homeserver erreicht werden.

Beispiel für DBeaver oder ähnliche Werkzeuge:

```text
Host: 192.168.x.x
Port: 5432
Database: postgres
User: postgres
Password: <Passwort aus .env>
```

Der Zugriff erfolgt nur im lokalen Heimnetz.

Es bestehen keine Portfreigaben ins Internet.

---

## Prüfung der migrierten Dienste

Nach der Migration wurden die Dienste im Heimnetz geprüft.

### Portainer

Portainer ist erreichbar über:

```text
https://192.168.x.x:9443
```

Die vorhandenen Portainer-Daten wurden über das Docker-Volume `portainer_data` vom Raspberry Pi 500 auf den Homeserver übernommen.

Nach der Wiederherstellung standen die bisherigen Portainer-Daten und Einstellungen auf dem neuen System wieder zur Verfügung.

### Uptime Kuma

Uptime Kuma ist erreichbar über:

```text
http://192.168.x.x:3001
```

Die vorhandenen Monitore wurden übernommen.

Einige Monitore mussten angepasst werden, da sie noch auf die alte IP-Adresse des Raspberry Pi 500 zeigten.

### Adminer

Adminer ist erreichbar über:

```text
http://192.168.x.x:8080
```

Der Zugriff auf PostgreSQL über den internen Servernamen `postgres` wurde geprüft.

### PostgreSQL

PostgreSQL ist über Port `5432` im lokalen Heimnetz erreichbar.

Der Dienst wurde für lokale Entwicklungs- und Administrationszwecke bereitgestellt.

---

## Ergebnis

Die bestehenden Homelab-Dienste wurden erfolgreich vom Raspberry Pi 500 auf den HP ProDesk 800 G5 Mini migriert.

Übernommen wurden:

* Portainer-Konfiguration
* Uptime-Kuma-Konfiguration
* PostgreSQL-Daten über SQL-Dump
* bestehende Dienststruktur im lokalen Heimnetz

Der neue Homeserver stellt die Dienste nun auf einer x86-Plattform mit Debian Server minimal und RAID-1-Speicherbasis bereit.

Die Migration wurde ohne Portfreigaben ins Internet durchgeführt.

---

## Erkenntnisse

* Docker-Volumes können gezielt über Archivdateien migriert werden.
* Bei einem Plattformwechsel von ARM auf x86 ist ein logischer PostgreSQL-Dump sinnvoller als das Kopieren des Datenbank-Volumes.
* Docker Compose erleichtert die Verwaltung mehrerer zusammengehöriger Dienste.
* Zugangsdaten sollten über `.env` ausgelagert und nicht in der Compose-Datei gespeichert werden.
* `docker compose config --quiet` prüft die Konfiguration, ohne sensible Werte auszugeben.
* Nach einer Migration müssen Monitoring-Ziele überprüft und gegebenenfalls an neue IP-Adressen angepasst werden.
* Dienste können im Heimnetz bereitgestellt werden, ohne sie per Portfreigabe aus dem Internet erreichbar zu machen.