# Monitoring und Betrieb des Homeservers

## Projektziel

Nach der Migration der Homelab-Dienste auf den neuen Homeserver soll der laufende Betrieb überprüfbar und nachvollziehbar dokumentiert werden.

Ziel ist es, den Zustand des Systems, der Docker-Dienste und der Hardware regelmäßig kontrollieren zu können.

Dabei sollen verschiedene Werkzeuge eingesetzt werden:

* Cockpit für System- und Hardwareüberblick
* Portainer für Docker-Verwaltung
* Uptime Kuma für Dienstüberwachung
* Adminer für PostgreSQL-Verwaltung
* Terminalbefehle für gezielte Prüfungen

---

## Ausgangssituation

Nach der Installation des Homeservers und der Migration der Docker-Dienste wurde der laufende Betrieb geprüft.

Im Fokus stehen jetzt nicht mehr Installation oder Migration, sondern:

* Erreichbarkeit der Dienste
* Zustand der Docker-Container
* Systemüberwachung des Debian-Hosts
* Temperatur- und Hardwarekontrolle
* laufende Betriebsprüfung im Heimnetz

Cockpit wurde dafür zusätzlich direkt auf dem Debian-Host installiert. Es ist kein migrierter Docker-Dienst, sondern dient der Systemüberwachung und Administration des Homeservers.

## Abgrenzung der Werkzeuge

Für den Betrieb des Homeservers werden mehrere Werkzeuge verwendet, die unterschiedliche Aufgaben erfüllen.

### Cockpit

Cockpit dient zur Überwachung und Verwaltung des Debian-Hosts.

Es zeigt unter anderem:

* Systemauslastung
* Speicherverbrauch
* Netzwerkaktivität
* laufende Dienste
* Systeminformationen
* Hardware- und Temperaturinformationen

Cockpit läuft direkt auf dem Debian-System und nicht als Docker-Container.

### Portainer

Portainer dient zur Verwaltung der Docker-Umgebung.

Darüber können Container, Images, Netzwerke und Volumes geprüft und verwaltet werden.

Portainer ist besonders hilfreich, um den Zustand der Container übersichtlich zu kontrollieren.

### Uptime Kuma

Uptime Kuma überwacht die Erreichbarkeit von Diensten.

Nach der Migration wurden die Monitore geprüft und bei Bedarf auf die neue lokale Adresse des Homeservers angepasst.

Damit kann schnell erkannt werden, ob ein Dienst im Heimnetz erreichbar ist.

### Adminer

Adminer dient zur webbasierten Verwaltung von PostgreSQL.

Adminer läuft als Docker-Container und verbindet sich intern mit dem PostgreSQL-Container über den Servicenamen `postgres`.

---

## Erreichbarkeit der Dienste

Die Dienste sind ausschließlich im lokalen Heimnetz erreichbar.

Beispielhafte lokale Adressen:

```text
Portainer:  https://192.168.x.x:9443
Uptime Kuma: http://192.168.x.x:3001
Adminer:    http://192.168.x.x:8080
Cockpit:    https://192.168.x.x:9090
PostgreSQL: 192.168.x.x:5432
```

Es bestehen keine Portfreigaben in der FRITZ!Box.

Die Dienste sind damit nicht direkt aus dem Internet erreichbar.

---

## Cockpit installieren

Cockpit wurde direkt auf dem Debian-Host installiert.

```bash
sudo apt update
sudo apt install cockpit
```

Nach der Installation wurde der Dienst aktiviert und gestartet.

```bash
sudo systemctl enable --now cockpit.socket
```

Der Status kann geprüft werden mit:

```bash
systemctl status cockpit.socket
```

Cockpit ist anschließend im lokalen Heimnetz erreichbar unter:

```text
https://192.168.x.x:9090
```

Beim ersten Aufruf kann der Browser eine Zertifikatswarnung anzeigen, da Cockpit standardmäßig ein lokal erzeugtes Zertifikat verwendet.

Für den lokalen Betrieb im Heimnetz ist das erwartbar.

---

## Docker-Dienste prüfen

Der Zustand der Docker-Container kann über das Terminal geprüft werden.

```bash
docker ps
```

Erwartete laufende Container:

* `portainer`
* `uptime-kuma`
* `postgres`
* `adminer`

Zusätzlich kann im Compose-Verzeichnis geprüft werden:

```bash
cd ~/docker/homelab
docker compose ps
```

Damit wird sichtbar, ob die Dienste des Compose-Projekts laufen.

---

## Docker-Compose-Projekt

Die Docker-Dienste werden im Verzeichnis `~/docker/homelab` verwaltet.

Der Hostname des Servers lautet `homeserver`.

Der Compose-Projektordner heißt `homelab`.

Damit sind Hostname und Compose-Projektname bewusst getrennt:

* `homeserver`: Name des Debian-Systems im Netzwerk
* `homelab`: Name des Docker-Compose-Projekts

Diese Trennung ist sinnvoll, weil der Server mehrere Aufgaben übernehmen kann, während das Compose-Projekt nur den aktuellen Homelab-Dienstestack beschreibt.

---

## Uptime-Kuma-Monitore

Nach der Migration mussten die Uptime-Kuma-Monitore geprüft werden.

Einige Monitore zeigten noch auf die alte IP-Adresse des Raspberry Pi 500.

Diese Ziele wurden auf die neue lokale Adresse des Homeservers angepasst.

Überwacht werden können beispielsweise:

* Portainer
* Adminer
* PostgreSQL
* lokale Webdienste
* später weitere Anwendungen

Uptime Kuma prüft dabei die Erreichbarkeit der Dienste, ersetzt aber keine vollständige Systemüberwachung.

---

## PostgreSQL prüfen

PostgreSQL läuft als Docker-Container und ist im lokalen Heimnetz über Port `5432` erreichbar.

Der Containerstatus kann geprüft werden mit:

```bash
docker ps | grep postgres
```

Eine einfache Prüfung innerhalb des Containers ist möglich mit:

```bash
docker exec -it postgres psql -U postgres -d postgres
```

Innerhalb von `psql` kann zum Beispiel geprüft werden:

```sql
\l
```

Damit werden die vorhandenen Datenbanken angezeigt.

Die Verbindung kann mit folgendem Befehl beendet werden:

```sql
\q
```

Das echte PostgreSQL-Passwort wird nicht in der Dokumentation gespeichert.

---

## Adminer prüfen

Adminer ist im lokalen Heimnetz erreichbar unter:

```text
http://192.168.x.x:8080
```

Für die Verbindung zu PostgreSQL wird innerhalb des Docker-Compose-Projekts der Servicename verwendet:

```text
Server: postgres
```

Das funktioniert, weil Adminer und PostgreSQL im gleichen Docker-Netzwerk laufen.

Beispielhafte Anmeldedaten:

```text
System: PostgreSQL
Server: postgres
User: postgres
Password: <Passwort aus .env>
Database: postgres
```

---

## NVMe-Informationen prüfen

Für die Prüfung der NVMe-SSDs kann `nvme-cli` verwendet werden.

Installation:

```bash
sudo apt install nvme-cli
```

Die vorhandenen NVMe-Geräte können angezeigt werden mit:

```bash
sudo nvme list
```

Weitere Informationen zu einer NVMe-SSD können abgefragt werden mit:

```bash
sudo nvme smart-log /dev/nvme0
sudo nvme smart-log /dev/nvme1
```

Dabei sind unter anderem Temperaturwerte und Gesundheitsinformationen sichtbar.

`nvme list` zeigt die physischen NVMe-SSDs, zum Beispiel `/dev/nvme0n1` und `/dev/nvme1n1`.

Der RAID-Verbund `/dev/md0` erscheint dort nicht, weil er kein physisches NVMe-Gerät ist, sondern ein logischer Linux-Software-RAID-Verbund.

Der RAID-Zustand wird deshalb separat geprüft:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```
---

## Temperaturkontrolle

Nach der Migration wurden die Temperaturen des Systems geprüft.

Die beobachteten Werte lagen im unkritischen Bereich.

Beispielhafte Werte im Betrieb:

```text
CPU: ca. 33 °C im Leerlauf
NVMe 1: ca. 35 °C
NVMe 2: ca. 30 °C
```
---

## Systemsensoren prüfen

Neben `nvme-cli` können auch Systemsensoren abgefragt werden.

Dafür kann `lm-sensors` installiert werden:

```bash
sudo apt install lm-sensors
```

Anschließend kann die Sensorerkennung gestartet werden:

```bash
sudo sensors-detect
```

Die aktuellen Sensorwerte können angezeigt werden mit:

```bash
sensors
```

Je nach Hardware und Kernelunterstützung werden unterschiedliche Sensoren angezeigt.

Nicht jeder Sensor eines Systems ist unter Linux immer vollständig auslesbar.

---

## Lüfterverhalten

Beim Betrieb des Homeservers wurde festgestellt, dass der Lüfter dauerhaft hörbar läuft.

Die Temperaturen von CPU und NVMe-SSDs lagen dabei trotzdem im unkritischen Bereich.

Im BIOS wurde die Einstellung für die minimale Lüfterdrehzahl geprüft.

Die Einstellung `Idle Fan Speed` stand auf `0`.

Trotzdem blieb der Lüfter im Betrieb wahrnehmbar.

Da die Temperaturen unauffällig sind, handelt es sich nicht um ein akutes thermisches Problem.

Ein möglicher nächster Schritt ist ein BIOS-Update, um zu prüfen, ob sich das Lüfterverhalten dadurch verbessert.

---

## BIOS-Update als nächster Schritt

Ein BIOS-Update wurde als möglicher weiterer Schritt vorgemerkt.

Ziel wäre nicht die Behebung eines akuten Fehlers, sondern die Prüfung, ob Firmware-Verbesserungen für Lüftersteuerung, Hardwarekompatibilität oder Systemstabilität verfügbar sind.

Vor einem BIOS-Update sollten folgende Punkte beachtet werden:

* genaue Modellbezeichnung prüfen
* aktuelle BIOS-Version notieren
* passende BIOS-Version vom Hersteller verwenden
* Update nicht während instabiler Stromversorgung durchführen
* nach dem Update BIOS-Einstellungen erneut kontrollieren

Das BIOS-Update ist damit kein zwingender Bestandteil der Migration, sondern ein sinnvoller späterer Wartungsschritt.

---

## Systemdienste prüfen

Laufende Systemdienste können mit `systemctl` geprüft werden.

Beispiel für Cockpit:

```bash
systemctl status cockpit.socket
```

Allgemein fehlgeschlagene Dienste können angezeigt werden mit:

```bash
systemctl --failed
```

Dieser Befehl ist hilfreich, um nach Neustarts oder Änderungen zu prüfen, ob Dienste fehlerfrei gestartet wurden.

---

## Speicherplatz prüfen

Der verfügbare Speicherplatz kann mit folgendem Befehl geprüft werden:

```bash
df -h
```

Der RAID-Verbund ist als Root-Dateisystem eingebunden.

Zusätzlich kann der RAID-Zustand geprüft werden mit:

```bash
cat /proc/mdstat
```

Ein gesunder RAID-1-Verbund mit zwei aktiven Laufwerken wird mit `[UU]` angezeigt.

---

## Sicherheit im laufenden Betrieb

Die Dienste laufen ausschließlich im lokalen Heimnetz.

Es wurden keine Portfreigaben in der FRITZ!Box eingerichtet.

Dadurch sind Portainer, Uptime Kuma, Adminer, Cockpit und PostgreSQL nicht direkt aus dem Internet erreichbar.

Zugangsdaten werden nicht in der Dokumentation gespeichert.

Die PostgreSQL-Zugangsdaten liegen in einer `.env`-Datei, die nicht in das Git-Repository aufgenommen wird.

Die Dateirechte der `.env`-Datei wurden eingeschränkt:

```bash
chmod 600 .env
```

---

## Bewertung des aktuellen Betriebs

Der aktuelle Zustand des Homeservers ist für das Homelab geeignet.

Die Dienste laufen auf einer stärkeren und besser erweiterbaren Plattform als zuvor auf dem Raspberry Pi 500.

Die wichtigsten Betriebsaspekte sind abgedeckt:

* Docker-Verwaltung über Portainer
* Dienstüberwachung über Uptime Kuma
* Systemüberwachung über Cockpit
* Datenbankverwaltung über Adminer
* gezielte technische Prüfung über Terminalbefehle
* Temperaturkontrolle der NVMe-SSDs
* RAID-Zustandsprüfung
* keine direkte Erreichbarkeit aus dem Internet

Damit ist der Homeserver nach der Migration betriebsbereit und nachvollziehbar dokumentiert.

---

## Ergebnis

Der Homeserver wurde nach der Migration erfolgreich in den laufenden Betrieb überführt.

Die wichtigsten Dienste sind erreichbar und können überwacht werden.

Cockpit ergänzt die bestehende Docker-Umgebung um eine direkte Systemübersicht des Debian-Hosts.

Portainer, Uptime Kuma, PostgreSQL und Adminer laufen weiterhin als Docker-Dienste im lokalen Heimnetz.

Die Hardwarewerte, insbesondere CPU- und NVMe-Temperaturen, liegen im unkritischen Bereich.

Das Lüfterverhalten wurde beobachtet und als möglicher Punkt für ein späteres BIOS-Update vorgemerkt.

---

## Erkenntnisse

* Nach einer Migration reicht es nicht aus, nur Container zu starten; Dienste, Daten, Monitoring und Systemzustand müssen geprüft werden.
* Cockpit ergänzt Portainer sinnvoll, weil es nicht Docker, sondern den Debian-Host überwacht.
* Uptime Kuma zeigt die Erreichbarkeit von Diensten, ersetzt aber keine vollständige System- oder Hardwareüberwachung.
* NVMe-Temperaturen können mit `nvme-cli` gezielt geprüft werden.
* Ein hörbarer Lüfter ist nicht automatisch ein thermisches Problem, wenn die Temperaturwerte unkritisch sind.
* RAID-Zustand, Speicherplatz und Systemdienste sollten regelmäßig kontrolliert werden.
* Die Trennung zwischen Host-System, Docker-Verwaltung, Dienstüberwachung und Datenbankverwaltung macht den Betrieb nachvollziehbarer.