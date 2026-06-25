# Docker installieren und testen

## Ziel

Docker soll auf dem Raspberry Pi 500 installiert werden, damit später Dienste wie Portainer, Uptime Kuma, Datenbanken und eigene Anwendungen als Container betrieben werden können.

---

## Schritt 1: Docker-Installationsskript herunterladen

### Durchführung

Die Installation wurde per SSH vom Mac aus auf dem Raspberry Pi durchgeführt.

Zuerst wurde das offizielle Docker-Installationsskript heruntergeladen:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

### Ergebnis

Die Datei `get-docker.sh` wurde auf dem Raspberry Pi gespeichert.

---

## Schritt 2: Docker installieren

### Durchführung

Das Installationsskript wurde mit Administratorrechten ausgeführt:

```bash
sudo sh get-docker.sh
```

### Bedeutung des Befehls

* `usermod`: ändert Benutzer-Einstellungen
* `-aG docker`: fügt den Benutzer zusätzlich zur Gruppe `docker` hinzu
* `$USER`: steht für den aktuell angemeldeten Benutzer

---

### Ergebnis

Docker wurde auf dem Raspberry Pi installiert.

---

## Schritt 3: Benutzer zur Docker-Gruppe hinzufügen

### Durchführung

Der Benutzer wurde zur Gruppe `docker` hinzugefügt:

```bash
sudo usermod -aG docker $USER
```

### Bedeutung

Dadurch können Docker-Befehle später ohne `sudo` ausgeführt werden.

### Hinweis

Der Befehl gibt bei erfolgreicher Ausführung keine eigene Erfolgsmeldung aus. Eine neue leere Eingabezeile bedeutet in diesem Fall, dass kein Fehler aufgetreten ist.

Damit die neue Gruppenzugehörigkeit aktiv wird, ist ein Neustart oder erneuter Login notwendig.

---

## Schritt 4: Neustart

### Durchführung

Der Raspberry Pi wurde neu gestartet:

```bash
sudo reboot
```

### Ergebnis

Die SSH-Verbindung wurde erwartungsgemäß beendet.

---

## Problem: SSH-Verbindung nach Neustart nicht direkt möglich

### Beschreibung

Nach dem Neustart war die vorherige SSH-Verbindung nicht mehr nutzbar.

### Ursache

Durch den Neustart und den Betrieb am Router konnte sich die IP-Adresse des Raspberry Pi ändern.

### Lösung

Die neue IP-Adresse wurde über die FRITZ!Box ermittelt:

```text
FRITZ!Box → Heimnetz → Netzwerk
```

Dort wurde der Raspberry Pi gesucht und die aktuelle IP-Adresse verwendet.

Anschließend wurde die SSH-Verbindung erneut hergestellt:

```bash
ssh <benutzername>@192.168.x.x
```

### Feste IP-Zuweisung über die FRITZ!Box

Damit die Dienste des Homelabs dauerhaft unter der gleichen lokalen Adresse erreichbar bleiben, wurde in der FRITZ!Box eine feste IPv4-Zuweisung für den Raspberry Pi eingerichtet.

Die IP-Adresse wird weiterhin per DHCP von der FRITZ!Box vergeben, bleibt durch die Reservierung im Router aber dauerhaft diesem Gerät zugeordnet.

Dadurch bleiben lokale Dienste wie Portainer, Uptime Kuma und Adminer zuverlässig über dieselbe Adresse erreichbar.

### Ergebnis

Die SSH-Verbindung funktioniert wieder. Durch die feste IPv4-Zuweisung in der FRITZ!Box bleibt der Raspberry Pi künftig unter derselben lokalen Adresse erreichbar.

---

## Schritt 5: Docker prüfen

### Durchführung

Nach dem erneuten Login wurde geprüft, ob Docker korrekt installiert wurde:

```bash
docker --version
```

Zusätzlich wurde die Gruppenzugehörigkeit geprüft:

```bash
groups
```

In der Ausgabe sollte `docker` enthalten sein.

---

## Schritt 6: Ersten Testcontainer starten

### Durchführung

Zum Funktionstest wurde der Docker-Testcontainer gestartet:

```bash
docker run hello-world
```

### Bedeutung des Befehls

* `docker run`: startet einen Container aus einem Image
* `hello-world`: offizielles Test-Image zur Prüfung der Docker-Installation

### Erwartetes Ergebnis

Wenn Docker korrekt funktioniert, erscheint eine Ausgabe mit:

```text
Hello from Docker!
```

### Ergebnis

Der Testcontainer wurde erfolgreich gestartet. Die Ausgabe enthielt:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Damit ist bestätigt, dass Docker auf dem Raspberry Pi 500 funktioniert und Container erfolgreich gestartet werden können.

---

## Erkenntnisse

* Docker ermöglicht das Ausführen isolierter Anwendungen in Containern.
* Die FRITZ!Box kann genutzt werden, um die aktuelle IP-Adresse des Raspberry Pi zu finden.
* Ein erfolgreicher `hello-world`-Container bestätigt, dass Docker grundsätzlich funktioniert.
