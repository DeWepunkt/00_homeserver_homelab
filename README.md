# Homeserver Homelab

## Projektziel

Aufbau eines lokalen Homeserver-Homelabs als Lern- und Entwicklungsumgebung für Linux, Docker, Netzwerke, Monitoring, Datenbanken und spätere eigene Softwareprojekte.

Das Projekt begann auf einem Raspberry Pi 500 und wurde später auf einen HP ProDesk 800 G5 Mini als x86-Homeserver migriert. Dadurch bleibt der ursprüngliche Lern- und Projektverlauf nachvollziehbar, während die aktuelle Plattform mehr Leistung, mehr Speicherplatz und mehr Flexibilität für den weiteren Ausbau bietet.

Das Homelab dient als praktische Plattform, um Anwendungen aus der Umschulung zum Fachinformatiker für Anwendungsentwicklung bereitzustellen, zu testen und nachvollziehbar zu dokumentieren.

---

## Entwicklung des Projekts

- Phase 1: Raspberry Pi 500 als erstes lokales Homelab
- Phase 2: Monitoring- und Datenbank-Grundlage mit Docker
- Phase 3: Migration auf HP ProDesk 800 G5 Mini als x86-Homeserver
- Phase 4: Ausbau als Plattform für eigene Softwareprojekte

## Aktueller Stand

Das Homelab läuft lokal im Heimnetz auf einem HP ProDesk 800 G5 Mini mit Debian Server minimal.

Die ursprüngliche Raspberry-Pi-500-Installation wurde als erste Projektphase dokumentiert und anschließend auf den neuen Homeserver migriert. Der Raspberry Pi 500 bleibt damit Teil der Projekthistorie, ist aber nicht mehr die aktuelle Hauptplattform.

Aktuell eingerichtet sind:

* Debian Server minimal im Headless-Betrieb
* SSH-Zugriff im Terminal
* Docker als Container-Plattform
* Portainer als Weboberfläche zur Docker-Verwaltung
* Uptime Kuma als Monitoring-Dienst
* PostgreSQL als Datenbankdienst
* Adminer als Weboberfläche zur Datenbankverwaltung
* Cockpit zur System- und Hardwareüberwachung

Die Dienste sind aktuell nicht über Portfreigaben aus dem Internet erreichbar.

---

## Hardware

### Aktuelle Plattform

* HP ProDesk 800 G5 Mini
* Intel Core i5, 9. Generation
* 32 GB RAM
* 2× 2 TB Lexar NM790 NVMe-SSDs
* RAID 1
* Debian Server minimal

### Ursprüngliche Plattform

* Raspberry Pi 500
* 8 GB RAM
* 128 GB microSD-Karte
* Raspberry Pi OS Lite

---

## Technische Entscheidungen

Der Raspberry Pi 500 wurde zu Beginn verwendet, weil er bereits vorhanden war, wenig Strom verbraucht und für ein kleines lokales Homelab mit Docker, Monitoring und Datenbankdiensten ausreichend Leistung bietet. Diese erste Phase diente dazu, Linux, SSH, Docker, Containerverwaltung und Monitoring praktisch aufzubauen und zu dokumentieren.

Die spätere Migration auf den HP ProDesk 800 G5 Mini wurde durchgeführt, weil die neue x86-Plattform mehr Leistungsreserven, mehr Arbeitsspeicher, bessere Erweiterbarkeit und eine geeignetere Grundlage für spätere Dienste und eigene Softwareprojekte bietet. Dadurch kann das Homelab langfristig als stabilere Entwicklungs- und Testumgebung genutzt werden.

Debian Server minimal wurde als aktuelles Betriebssystem gewählt, weil es ohne grafische Oberfläche betrieben wird, wenig Ressourcen verbraucht und sich gut für einen Headless-Homeserver eignet. Die Verwaltung erfolgt hauptsächlich per SSH vom Mac aus. Ergänzend wird Cockpit zur System- und Hardwareüberwachung genutzt.

Die beiden 2-TB-NVMe-SSDs werden im RAID 1 betrieben. Dadurch steht nicht die maximale Gesamtkapazität im Vordergrund, sondern eine höhere Ausfallsicherheit bei einem einzelnen SSD-Defekt. Das ersetzt kein Backup, erhöht aber die Stabilität der lokalen Homeserver-Plattform.

Docker dient als Grundlage, um einzelne Dienste isoliert und nachvollziehbar als Container zu betreiben. Dadurch können Portainer, Uptime Kuma, PostgreSQL und Adminer getrennt voneinander gestartet, gestoppt und dokumentiert werden.

Ein weiterer Vorteil ist, dass neue Dienste vergleichsweise einfach ergänzt oder bestehende Dienste wieder entfernt werden können, ohne das Grundsystem direkt zu verändern. Die einzelnen Container bleiben dadurch weitgehend unabhängig voneinander verwaltbar.

Portainer wurde eingerichtet, um Docker-Container zusätzlich über eine Weboberfläche verwalten zu können. Das erleichtert die Übersicht über laufende Container, Images, Volumes und Netzwerke.

Uptime Kuma wurde gewählt, um die Erreichbarkeit der Homelab-Dienste zu überwachen. Dadurch wird sichtbar, ob der Homeserver und die wichtigsten Dienste wie Portainer, Adminer und PostgreSQL erreichbar sind.

PostgreSQL wurde als Datenbankdienst eingerichtet, weil spätere Java-Anwendungen nicht nur lokal mit SQLite, sondern auch mit einem externen Datenbankdienst betrieben werden können.

Adminer dient als einfache Weboberfläche, um PostgreSQL im Browser zu prüfen und zu verwalten.

---

## Software & Werkzeuge

### Aktuell verwendet

* Debian Server minimal
* SSH für den Fernzugriff
* Git für die Versionsverwaltung
* Docker
* Portainer
* Uptime Kuma
* PostgreSQL
* Adminer
* Cockpit
* KI-gestützte Unterstützung bei Dokumentation und Strukturierung

### In der ursprünglichen Raspberry-Pi-Phase verwendet

* Raspberry Pi OS Lite 64-bit

---

## Dienste und lokale Adressen

* **Portainer**  
  Docker-Verwaltung per Weboberfläche  
  Zugriff: `https://192.168.x.x:9443`

* **Uptime Kuma**  
  Monitoring der Homelab-Dienste  
  Zugriff: `http://192.168.x.x:3001`

* **Adminer**  
  Weboberfläche zur Datenbankverwaltung  
  Zugriff: `http://192.168.x.x:8080`

* **PostgreSQL**  
  Datenbankdienst  
  Zugriff: je nach Projektphase intern im Docker-Netzwerk oder lokal im Heimnetz über Port `5432`

Die Dienste laufen im lokalen Heimnetz und sind nicht über Portfreigaben aus dem Internet erreichbar.

---

## Projektfortschritt

### Phase 1: Basis-Homelab auf dem Raspberry Pi 500

* [x] Raspberry Pi OS Lite installiert
* [x] SSH vorbereitet
* [x] Netzwerkverbindung hergestellt
* [x] SSH-Verbindung vom Mac erfolgreich getestet
* [x] System aktualisiert
* [x] Headless-Betrieb am Router getestet
* [x] Grundwerkzeuge installiert
* [x] Docker installiert
* [x] Ersten Container gestartet
* [x] Portainer eingerichtet

### Phase 2: Monitoring-Grundlage

* [x] Uptime Kuma eingerichtet
* [x] Portainer-Monitor eingerichtet
* [x] Raspberry-Pi-Ping-Monitor eingerichtet

### Phase 3: Datenbank-Grundlage

* [x] Docker-Netzwerk `homelab-net` eingerichtet
* [x] PostgreSQL-Container eingerichtet
* [x] Adminer-Container eingerichtet
* [x] Anmeldung an PostgreSQL über Adminer erfolgreich getestet
* [x] Testtabelle in PostgreSQL erstellt
* [x] Testdaten eingefügt und erfolgreich abgefragt
* [x] Adminer in Uptime Kuma überwacht
* [x] PostgreSQL intern per TCP-Port-Monitor in Uptime Kuma überwacht

### Phase 4: Migration auf HP ProDesk 800 G5 Mini

* [x] HP ProDesk 800 G5 Mini als neue Homeserver-Plattform vorbereitet
* [x] Debian Server minimal installiert
* [x] Headless-Betrieb eingerichtet
* [x] SSH-Zugriff auf dem neuen Homeserver eingerichtet und getestet
* [x] Zwei 2-TB-NVMe-SSDs eingebaut
* [x] RAID 1 als Speichergrundlage eingerichtet
* [x] Cockpit zur System- und Hardwareüberwachung eingerichtet
* [x] Hardware des neuen Homeservers dokumentiert
* [x] Projektbeschreibung von Raspberry-Pi-Homelab auf Homeserver-Homelab erweitert

---

## Verzeichnisstruktur

```text
homeserver-homelab/
├── README.md
├── .gitignore
├── docs/
│   ├── 01-installation.md
│   ├── 02-docker.md
│   ├── 03-portainer.md
│   ├── 04-uptime-kuma.md
│   ├── 05-postgresql-adminer.md
│   └── 06-hardware-homeserver.md
└── images/
    ├── 03-portainer-dashboard.png
    ├── 04-uptime-kuma-dashboard.png
    ├── 05-postgresql-adminer.png
    ├── 05-uptime-kuma-dashboard-postgresql-adminer.png
    ├── 06-hardware-homeserver.png
    └── 06-hardware-homeserver-NVMe.png
```

---

## Dokumentation

* Installation und Ersteinrichtung auf dem Raspberry Pi 500: `docs/01-installation.md`
* Docker-Einrichtung auf dem Raspberry Pi 500: `docs/02-docker.md`
* Portainer-Einrichtung: `docs/03-portainer.md`
* Monitoring mit Uptime Kuma: `docs/04-uptime-kuma.md`
* PostgreSQL und Adminer: `docs/05-postgresql-adminer.md`
* Hardware des neuen Homeservers: `docs/06-hardware-homeserver.md`

Die Detaildokumentation enthält die jeweiligen Schritte, Befehle, Probleme, Ursachen und Lösungen. Die README dient bewusst nur als Projektübersicht.

---

## Sicherheit

Die eingerichteten Dienste laufen aktuell nur im lokalen Heimnetz.

In der FRITZ!Box sind keine Portfreigaben für den Homeserver eingerichtet. Damit sind Portainer, Uptime Kuma, Adminer und PostgreSQL nicht direkt aus dem Internet erreichbar.

Zusätzlich gilt:

* Für Portainer, Uptime Kuma, Adminer und PostgreSQL werden eigene Passwörter verwendet.
* Das echte PostgreSQL-Passwort wird nicht in der Dokumentation gespeichert.
* In der Testdatenbank werden keine echten privaten Daten gespeichert.
* Externe Zugänge wie Tailscale, Cloudflare Tunnel, VPN oder Portfreigaben werden aktuell nicht genutzt.

PostgreSQL kann im Heimnetz für lokale Entwicklungs- und Administrationszwecke erreichbar gemacht werden. Der Dienst ist jedoch nicht über das Internet veröffentlicht.

---

## Versionsverwaltung

Das Projekt wird lokal mit Git versioniert. Änderungen werden in einzelnen Commits festgehalten, damit der Projektfortschritt nachvollziehbar bleibt.

Der erste Commit dokumentiert die abgeschlossene Ersteinrichtung des Raspberry Pi 500 Homelabs mit Raspberry Pi OS Lite, SSH, Docker, erstem Testcontainer und Portainer.

Spätere Commits dokumentieren die Erweiterung des Projekts um Monitoring, PostgreSQL, Adminer sowie die Migration auf den HP ProDesk 800 G5 Mini als aktuelle Homeserver-Plattform.

Das Repository wurde vom ursprünglichen Raspberry-Pi-Homelab zu einem allgemeineren Homeserver-Homelab weiterentwickelt. Dadurch bleibt die erste Projektphase weiterhin nachvollziehbar, während der aktuelle Stand korrekt als HP-ProDesk-Homeserver dokumentiert ist.

---

## Nächste Schritte

* BIOS-Update zur möglichen Optimierung der Lüftersteuerung prüfen
* eigene Java-Anwendung als separates Projekt vorbereiten
* Deployment der Java-Anwendung auf dem Homeserver planen
* PostgreSQL-Anbindung der Java-Anwendung umsetzen
* Später: lokale Cloud-/Dateiserver-Funktion im Heimnetz evaluieren