# Debian-Installation mit RAID 1

## Projektziel

Der neue Homeserver soll mit Debian Server minimal eingerichtet werden.

Die Installation erfolgt direkt auf zwei identischen 2-TB-NVMe-SSDs. Ziel ist ein RAID-1-Setup, bei dem das System auf einem gespiegelten Speicherverbund läuft.

Dadurch soll der Homeserver eine stabile Grundlage für Docker-Dienste, Datenbanken, Monitoring und spätere lokale Dienste erhalten.

---

## Ausgangssituation

Als neue Hardware wird ein HP ProDesk 800 G5 Mini eingesetzt.

Die Speicherbasis besteht aus zwei identischen 2-TB-Lexar-NM790-NVMe-SSDs.

Die Migration auf den neuen Homeserver wurde bewusst vor weiteren Erweiterungen durchgeführt. Dadurch kann die neue Speicher- und Betriebssystemstruktur direkt sauber aufgebaut werden, bevor zusätzliche Dienste oder produktive Daten hinzukommen.

---

## Entscheidung für Debian Server minimal

Als Betriebssystem wurde Debian ohne grafische Desktop-Umgebung installiert.

Debian wurde gewählt, weil es für Serverumgebungen gut geeignet ist:

* stabile Paketbasis
* geringe Grundlast ohne Desktop-Oberfläche
* gute Unterstützung für Docker und Serverdienste
* breite Dokumentation
* langfristig gut wartbar
* geeignet für Headless-Betrieb per SSH

Während der Installation wurde keine grafische Desktop-Umgebung ausgewählt. Installiert wurden nur die notwendigen Grundkomponenten für ein schlankes Serversystem.

Ausgewählt wurden:

* SSH-Server
* Standard-Systemwerkzeuge

Nicht ausgewählt wurden:

* Desktop-Umgebung
* Webserver
* zusätzliche grafische Komponenten

---

## Grundlegende Installationsentscheidungen

Während der Debian-Installation wurden folgende Grundentscheidungen getroffen:

* Hostname: `homeserver`
* Domain: leer gelassen
* kein separates root-Passwort vergeben
* administrativer Benutzer mit sudo-Rechten
* Debian-Spiegelserver aus Deutschland
* kein HTTP-Proxy
* Teilnahme an `popularity-contest`: nein

Da kein separates root-Passwort gesetzt wurde, erfolgt die Administration nicht über eine direkte root-Anmeldung. Administrative Aufgaben werden stattdessen über den normalen Benutzer mit `sudo` ausgeführt.

Das ist für einen Serverbetrieb sinnvoll, da der direkte root-Zugang dadurch nicht als regulärer Login verwendet wird.

---

## Netzwerk

Die Netzwerkkonfiguration des Homeservers erfolgt nicht über eine fest im Betriebssystem eingetragene statische IP-Adresse.

Stattdessen wird die IP-Adresse zentral über die FRITZ!Box verwaltet. Dort wurde für den Homeserver eine feste IPv4-Zuweisung eingerichtet.

Die Adresse wird weiterhin per DHCP vergeben, bleibt durch die Reservierung im Router aber dauerhaft dem Homeserver zugeordnet.

Vorteile dieser Lösung:

* zentrale Verwaltung im Router
* keine manuelle Netzwerkkonfiguration im Debian-System notwendig
* geringeres Risiko von IP-Konflikten
* Dienste bleiben im Heimnetz unter derselben lokalen Adresse erreichbar

---

## Entscheidung für RAID 1

Für den Homeserver wurde ein RAID-1-Setup gewählt.

Bei RAID 1 werden Daten gespiegelt auf zwei Laufwerken gespeichert. Die nutzbare Kapazität entspricht ungefähr der Kapazität einer einzelnen SSD, während die zweite SSD als Spiegel dient.

In diesem Projekt bedeutet das:

* 2× 2 TB physische SSDs
* ca. 2 TB nutzbare Kapazität
* Spiegelung der Daten auf beide Laufwerke
* höhere Verfügbarkeit bei Ausfall einer einzelnen SSD

Der Vorteil besteht darin, dass eine SSD ausfallen kann, ohne dass das System unmittelbar verloren ist.

Dem steht als Nachteil gegenüber, dass nur etwa die Hälfte der physisch vorhandenen Speicherkapazität nutzbar ist.

Wichtig ist die Abgrenzung:

* RAID 1 erhöht die Verfügbarkeit.
* RAID 1 ersetzt kein Backup.

Fehlerhafte Änderungen, versehentlich gelöschte Dateien oder beschädigte Daten würden ebenfalls auf beide SSDs gespiegelt. Für spätere produktive Daten ist daher weiterhin ein separates Backup-Konzept erforderlich.

---

## Partitionierung

Die Partitionierung wurde während der Debian-Installation manuell eingerichtet.

Auf beiden NVMe-SSDs wurde jeweils eine EFI-Systempartition erstellt. Zusätzlich wurde der verbleibende Speicherbereich beider SSDs für Linux Software RAID vorbereitet.

Die grundlegende Struktur:

* SSD 1:
  * EFI-Systempartition
  * Partition für Linux Software RAID
* SSD 2:
  * EFI-Systempartition
  * Partition für Linux Software RAID

Die EFI-Partitionen liegen bewusst außerhalb des RAID-Verbunds.

Der Grund dafür ist, dass UEFI bereits vor dem Start von Linux auf die Bootdateien zugreifen muss. Das Linux-Software-RAID über `mdadm` steht zu diesem Zeitpunkt noch nicht zur Verfügung.

Deshalb erhält jede SSD eine eigene EFI-Systempartition.

---

## EFI-Systempartitionen

Für jede SSD wurde eine eigene EFI-Systempartition angelegt.

Die Größe wurde im Installer mit 512 MiB gewählt. Je nach Anzeige kann dies im Installer als ungefähr 535 MB dargestellt werden.

Der Unterschied entsteht durch unterschiedliche Einheiten:

* MiB: binäre Einheit
* MB: dezimale Einheit

Die abweichende Anzeige ist daher normal und kein Fehler.

Die EFI-Partitionen wurden nicht in das RAID aufgenommen.

---

## RAID-Partitionen

Der jeweils große verbleibende Speicherbereich beider SSDs wurde als physisches Volume für Linux Software RAID markiert.

Die beteiligten Partitionen sind:

```text
/dev/nvme0n1p2
/dev/nvme1n1p2
```

Aus diesen beiden Partitionen wurde ein RAID-1-Verbund erstellt.

Der RAID-Verbund wurde als `/dev/md0` angelegt.

Konfiguration:

* RAID-Level: RAID 1
* aktive Geräte: 2
* Ersatzgeräte: 0
* Gerät: `/dev/md0`

---

## Dateisystem und Einbindung

Der RAID-Verbund `/dev/md0` wurde als ext4-Dateisystem formatiert.

Das Dateisystem wurde als Root-Dateisystem eingebunden:

```text
Mountpoint: /
Dateisystem: ext4
Gerät: /dev/md0
```

Damit liegt das Debian-System selbst auf dem RAID-1-Verbund.

---

## Reservierte Blöcke bei ext4

Bei ext4 werden standardmäßig reservierte Blöcke für administrative Zwecke zurückgehalten. Auf großen Dateisystemen kann der Standardwert von 5 % jedoch sehr viel Speicher belegen.

Bei einem Speicherbereich von ungefähr 2 TB wären 5 % etwa 100 GB.

Deshalb wurde der Anteil der reservierten Blöcke auf 1 % reduziert.

Damit bleibt weiterhin ein administrativer Reservebereich erhalten, ohne unnötig viel Speicherplatz zu blockieren.

Die Anpassung kann nach der Installation mit folgendem Befehl erfolgen:

```bash
sudo tune2fs -m 1 /dev/md0
```

---

## Kein Swap-Bereich

Während der Installation wurde kein separater Swap-Bereich angelegt.

Der Homeserver verfügt über 32 GB RAM. Für den aktuellen Einsatzzweck mit Debian, Docker-Diensten, Monitoring und Datenbank ist das ausreichend.

Falls später Swap benötigt wird, kann nachträglich eine Swap-Datei auf dem bestehenden Dateisystem angelegt werden.

Der Verzicht auf eine separate Swap-Partition hält die Partitionierung einfacher und flexibler.

---

## Abschluss der Installation

Nach Abschluss der Installation wurde das System neu gestartet.

Der erste Zugriff erfolgte per SSH im lokalen Heimnetz.

Beispiel:

```bash
ssh <benutzername>@192.168.x.x
```

Beim ersten Verbindungsaufbau wurde der SSH-Fingerprint geprüft und bestätigt.

Dass bei der Passworteingabe im Terminal keine Zeichen angezeigt werden, ist normal. Linux zeigt bei Passwortabfragen im Terminal üblicherweise weder Zeichen noch Platzhalter an.

---

## RAID-Synchronisation

Nach der Installation wurde der RAID-1-Verbund initial synchronisiert.

Diese Synchronisation kann einige Zeit dauern, auch wenn auf dem System noch kaum Daten liegen. Der Grund ist, dass der RAID-Verbund blockweise über den gesamten vorgesehenen Speicherbereich synchronisiert wird.

Während dieser Synchronisation sollte das System nicht erzwungen ausgeschaltet werden.

Beim Versuch, das System während der RAID-Synchronisation auszuschalten, kann systemd den Vorgang blockieren, weil noch ein mdraid-Synchronisationsjob läuft.

In diesem Projekt wurde der Vorgang nicht erzwungen abgebrochen. Stattdessen wurde gewartet, bis die RAID-Synchronisation abgeschlossen war.

Das ist sinnvoll, um die Integrität des RAID-Verbunds nicht unnötig zu gefährden.

---

## RAID-Status prüfen

Der aktuelle Status des RAID-Verbunds kann mit folgendem Befehl geprüft werden:

```bash
cat /proc/mdstat
```

Ein gesunder RAID-1-Verbund mit zwei aktiven Laufwerken wird beispielsweise mit `[UU]` angezeigt.

Dabei bedeutet:

* `U`: Laufwerk aktiv und synchron
* `[UU]`: beide Laufwerke aktiv und synchron
* `[_U]` oder `[U_]`: ein Laufwerk fehlt oder ist nicht aktiv

---

## RAID-Details prüfen

Weitere Details zum RAID-Verbund können mit `mdadm` angezeigt werden:

```bash
sudo mdadm --detail /dev/md0
```

Wichtige Punkte in der Ausgabe sind:

* RAID-Level
* Anzahl der aktiven Geräte
* Zustand des Verbunds
* beteiligte Partitionen
* Synchronisationsstatus

Für dieses Projekt ist besonders wichtig, dass beide NVMe-Partitionen als aktiv angezeigt werden.

---

## RAID-Konfiguration prüfen

Damit der RAID-Verbund beim Systemstart zuverlässig erkannt wird, muss die Konfiguration in `mdadm.conf` hinterlegt sein.

Die erkannte RAID-Konfiguration kann mit folgendem Befehl angezeigt werden:

```bash
sudo mdadm --detail --scan
```

Die gespeicherte Konfiguration kann geprüft werden mit:

```bash
grep -E "^ARRAY" /etc/mdadm/mdadm.conf
```

Wenn dort ein Eintrag für `/dev/md0` vorhanden ist, ist die RAID-Konfiguration dauerhaft hinterlegt.

---

## Ergebnis

Debian Server minimal wurde erfolgreich auf dem HP ProDesk 800 G5 Mini installiert.

Das System läuft auf einem RAID-1-Verbund aus zwei identischen 2-TB-NVMe-SSDs.

Die wichtigsten Eigenschaften des Setups:

* Debian ohne Desktop-Umgebung
* Headless-Betrieb
* SSH-Zugriff im Heimnetz
* zwei separate EFI-Systempartitionen
* Linux Software RAID 1 mit `mdadm`
* `/dev/md0` als ext4-Dateisystem
* Root-Dateisystem auf dem RAID-Verbund
* reduzierte ext4-Reserveblöcke
* kein separater Swap-Bereich
* IP-Verwaltung über DHCP-Reservierung in der FRITZ!Box

Damit steht eine stabile Grundlage für die weitere Einrichtung des Homeservers zur Verfügung.

---

## Erkenntnisse

* Debian eignet sich gut als schlanke Grundlage für einen Homeserver.
* Eine Serverinstallation ohne Desktop reduziert unnötige Komponenten.
* RAID 1 erhöht die Verfügbarkeit bei Ausfall einer SSD, ersetzt aber kein Backup.
* EFI-Systempartitionen liegen bei diesem Setup bewusst außerhalb des Linux-Software-RAIDs.
* Zwei identische NVMe-SSDs sind eine sinnvolle Grundlage für ein RAID-1-System.
* Die RAID-Synchronisation kann auch bei einem frisch installierten System längere Zeit dauern.
* Während einer laufenden RAID-Synchronisation sollte das System nicht erzwungen ausgeschaltet werden.
* Eine DHCP-Reservierung in der FRITZ!Box ist für ein Heimnetz oft einfacher zu verwalten als eine statische IP-Konfiguration im Betriebssystem.