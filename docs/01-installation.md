# Installation und Ersteinrichtung

## Ziel

Einrichtung eines Raspberry Pi 500 als Grundlage für ein Homelab mit Linux, Docker und späteren Softwareprojekten.

---

## Schritt 1: Raspberry Pi OS Lite installieren

### Durchführung

Mit dem Raspberry Pi Imager wurde Raspberry Pi OS Lite 64-bit auf die microSD-Karte geschrieben.

Folgende Einstellungen wurden bereits während des Flash-Vorgangs vorgenommen:

* Hostname: `pi500-homelab`
* Benutzerkonto angelegt
* SSH aktiviert
* WLAN konfiguriert
* Deutsche Zeitzone und Tastaturbelegung gewählt

### Ergebnis

Der Raspberry Pi konnte erfolgreich von der microSD-Karte starten.

---

## Schritt 2: Netzwerkverbindung und SSH prüfen

### Ziel

Der Raspberry Pi soll im Heimnetz erreichbar sein und später ohne Monitor verwaltet werden können.

### Erster Verbindungsversuch

Vom Mac aus wurde versucht, eine SSH-Verbindung herzustellen:

```bash
ssh <benutzername>@pi500-homelab.local
```

### Problem

Die Verbindung schlug fehl:

```text
Operation timed out
```

Zusätzlich ergab ein Ping-Test:

```text
Host is down
```

### Analyse

Der Raspberry Pi war nicht korrekt im Netzwerk erreichbar.

Die Netzwerkkonfiguration wurde direkt am Raspberry Pi überprüft.

Bei der ersten Prüfung lieferte der Befehl

```bash
hostname -I
```

keine IP-Adresse zurück.

### Ursache

Der Raspberry Pi hatte keine aktive Netzwerkverbindung und erhielt deshalb keine IP-Adresse.

### Lösung

Die Netzwerkverbindung wurde direkt am Raspberry Pi überprüft und erneut hergestellt.

Zur Kontrolle wurde die Internetverbindung getestet:

```bash
ping -c 4 google.de
```

Anschließend wurde die IP-Adresse erneut abgefragt:

```bash
hostname -I
```

Nun wurde eine gültige IP-Adresse angezeigt.

### Ergebnis

Der Raspberry Pi ist im Heimnetz erreichbar und kann per SSH verwaltet werden.

## Erkenntnisse

* Raspberry Pi OS Lite eignet sich für den Headless-Betrieb.
* SSH sollte bereits beim Flashen aktiviert werden.
* `hostname -I` eignet sich zur schnellen Kontrolle der Netzwerkverbindung.
* `ping -c 4` ist hilfreich zur Überprüfung der Internetverbindung.

---

## Schritt 3: System aktualisieren

### Ziel

Das frisch installierte Raspberry Pi OS sollte auf den aktuellen Stand gebracht werden, bevor weitere Software installiert wird.

### Durchführung

Die Aktualisierung erfolgte per SSH-Verbindung auf dem Raspberry Pi.

Zunächst wurden die Paketinformationen aktualisiert:

```bash
sudo apt update
```

Anschließend wurden alle verfügbaren Aktualisierungen installiert:

```bash
sudo apt full-upgrade -y
```

Nach Abschluss der Installation wurde das System neu gestartet:

```bash
sudo reboot
```

### Ergebnis

Der Raspberry Pi läuft nun mit aktuellen Paketständen und bildet eine saubere Grundlage für die weitere Einrichtung des Homelabs.

### Erkenntnisse

* Systemupdates sollten direkt nach einer Neuinstallation durchgeführt werden.
* Der Neustart stellt sicher, dass alle Änderungen sauber übernommen werden.
* Die Administration kann vollständig über SSH erfolgen.

---

## Schritt 4: Grundwerkzeuge installieren

### Ziel

Für die weitere Arbeit am Homelab wurden grundlegende Werkzeuge installiert, die für Versionsverwaltung, spätere Installationen, Systemüberwachung und Firewall-Konfiguration benötigt werden.

### Durchführung

Die Installation erfolgte per SSH auf dem Raspberry Pi:

```bash
sudo apt install git curl htop ufw -y
```

### Bedeutung der Werkzeuge

* `git`: Versionsverwaltung und spätere Verbindung zu GitHub
* `curl`: Abrufen von Dateien und Installationsskripten
* `htop`: Anzeige der Systemauslastung
* `ufw`: Werkzeug zur späteren Firewall-Konfiguration, wurde installiert aber noch nicht aktiviert

### Ergebnis

Die Grundwerkzeuge sind installiert. Der Raspberry Pi ist damit für die weitere Einrichtung des Homelabs vorbereitet.
