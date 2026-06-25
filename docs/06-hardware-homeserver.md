# Hardware des neuen Homeservers

## Projektziel

Der Raspberry Pi 500 soll durch einen leistungsfähigeren und flexibleren Homeserver ersetzt werden.

Die Migration erfolgt als vorausschauender Schritt für den weiteren Ausbau des Homelabs.

---

## Ausgangssituation

Das Homelab wurde ursprünglich auf einem bereits vorhandenen Raspberry Pi 500 eingerichtet.

Auf dem Raspberry Pi 500 liefen bereits mehrere Dienste:

* Docker
* Portainer
* Uptime Kuma
* PostgreSQL
* Adminer

Für diesen Anwendungszweck war der Raspberry Pi 500 nicht überlastet und hatte weiterhin Reserven.

Der Wechsel auf eine neue Plattform erfolgte daher nicht aus einem akuten Leistungsproblem heraus, sondern aus folgenden Gründen:

* frühzeitige Migration vor weiteren Erweiterungen
* flexiblere x86-Plattform
* interne NVMe-Speicheranbindung
* bessere Erweiterbarkeit gegenüber einem Single Board Computer
* sinnvolle Weiterverwendung vorhandener Hardware
* Vorbereitung auf spätere lokale Dienste und eigene Anwendungen

---

## Neue Hardware

Als neue Plattform wurde ein HP ProDesk 800 G5 Mini eingesetzt.

Die Hardware des neuen Homeservers:

* Gerät: HP ProDesk 800 G5 Mini
* CPU: Intel Core i5, 9. Generation
* RAM: 32 GB
* Speicher: 2× 2 TB Lexar NM790 NVMe-SSDs
* Betriebssystem: Debian Server minimal
* Hostname: `homeserver`

Der Server wird headless betrieben. Monitor, Tastatur und Maus werden nur für Wartungsschritte wie BIOS-Einstellungen benötigt.

Die Verwaltung erfolgt hauptsächlich per SSH im Terminal. Ergänzend stellen einzelne Dienste eigene Weboberflächen bereit, zum Beispiel Portainer für Docker, Uptime Kuma für Monitoring und Adminer für PostgreSQL.

---

## Begründung der Plattformwahl

Der Raspberry Pi 500 ist als Single Board Computer für viele Homelab-Aufgaben gut geeignet.

Für den geplanten weiteren Ausbau bietet ein kompakter x86-Business-PC jedoch mehr Flexibilität und eine breitere Erweiterbarkeit.

Der HP ProDesk 800 G5 Mini wurde aus mehreren Gründen gewählt:

* x86-Plattform mit breiter Softwareunterstützung
* zwei interne NVMe-Steckplätze
* deutlich höherer Datendurchsatz als typische microSD- oder USB-basierte SBC-Setups
* mehr Arbeitsspeicherreserven
* reaktionsfreudigeres Systemverhalten
* kompakte Bauform
* wartbare Business-Hardware
* vergleichsweise niedriger Energiebedarf

Zusätzlich war das Gerät im Verhältnis zu aktuellen Angeboten vergleichbarer kompakter Business-Systeme wirtschaftlich attraktiv. Besonders kompakte Systeme mit 16 GB oder 32 GB RAM werden häufig deutlich teurer angeboten.

In Kombination mit den zwei internen NVMe-Steckplätzen eignet sich der HP ProDesk 800 G5 Mini gut als Grundlage für einen Homeserver.

---

## Speicherentscheidung

Im Homeserver werden zwei identische 2-TB-Lexar-NM790-NVMe-SSDs eingesetzt.

Beide Laufwerke haben:

* dieselbe Baureihe
* dieselbe Modellbezeichnung
* dieselbe Kapazität
* vergleichbare Leistungsdaten

Die Entscheidung für diese SSDs entstand aus vorhandener Hardware.

Eine der 2-TB-SSDs wurde aus einem externen SSD-Gehäuse übernommen, da dieses Laufwerk nicht aktiv genutzt wurde.

Die zweite 2-TB-SSD wurde aus einer mobilen Workstation ausgebaut. Aufgrund des geringen lokalen Speicherbedarfs wurde der interne Speicher der mobilen Workstation bewusst auf 1 TB reduziert. Die dafür verwendete 1-TB-SSD war bereits aus einem anderen bestehenden System vorhanden.

Dadurch konnte die größere 2-TB-SSD im Homeserver sinnvoller eingesetzt werden.

Die vorhandene Speicherhardware wird damit wie folgt genutzt:

* Mobile Workstation: 1 TB interner Speicher
* Homeserver: 2× 2 TB Lexar NM790 NVMe-SSDs
* Externes Laufwerk: 256-GB-NVMe-SSD im externen Gehäuse

Gerade bei aktuell hohen Speicherpreisen ist die sinnvolle Weiterverwendung vorhandener Komponenten wirtschaftlich und effizient.

---

## RAID-1-Grundlage

Die beiden identischen 2-TB-SSDs bilden die Grundlage für ein RAID-1-Setup.

Bei RAID 1 werden Daten gespiegelt gespeichert. Die nutzbare Kapazität entspricht ungefähr der Kapazität einer einzelnen SSD, während die zweite SSD als Spiegel dient.

Der wesentliche Vorteil ist die höhere Ausfallsicherheit. Eine SSD kann ausfallen, ohne dass das System unmittelbar verloren ist.

Dem steht als Nachteil gegenüber, dass sich die nutzbare Speicherkapazität im Vergleich zu zwei einzeln genutzten SSDs halbiert.

Für dieses Projekt überwiegen die Vorteile:

* höhere Ausfallsicherheit bei einem einzelnen Laufwerksausfall
* sinnvolle Nutzung zweier identischer SSDs
* ca. 2 TB nutzbare Kapazität für System, Dienste und spätere Daten
* bessere Grundlage für dauerhaft betriebene Dienste

Wichtig ist die fachliche Abgrenzung:

* RAID 1 erhöht die Verfügbarkeit bei Ausfall einer SSD.
* RAID 1 ersetzt kein Backup.

Da aktuell keine exklusiven Produktivdaten ausschließlich auf dem Homeserver liegen, wurde zunächst kein vollständiges Backup-Konzept umgesetzt. Für spätere lokale Cloud-, Dateiserver- oder Backupfunktionen wird das Thema erneut bewertet.

---

## Weiterverwendung der ursprünglichen SSD

Die ursprünglich im HP ProDesk verbaute 256-GB-NVMe-SSD wird als externes Laufwerk weiterverwendet.

Dafür wurde sie in ein externes NVMe-Gehäuse übernommen.

Damit bleibt sie flexibel nutzbar und bietet eine ausreichende mobile Speicherreserve für andere Geräte. Da die vorhandenen Arbeitsgeräte aktuell keinen hohen lokalen Speicherbedarf haben, ist diese Kapazität für den vorgesehenen Zweck ausreichend.

---

## Physischer Aufbau

Die beiden 2-TB-NVMe-SSDs wurden im HP ProDesk 800 G5 Mini verbaut und bilden die Speicherbasis des neuen Homeservers.

![Geöffneter HP ProDesk 800 G5 Mini](../images/06-hardware-homeserver.png)

*Geöffneter HP ProDesk 800 G5 Mini mit eingebauten NVMe-SSDs.*

![Detailansicht der eingebauten NVMe-SSDs](../images/06-hardware-homeserver-NVMe.png)

*Detailansicht der beiden Lexar-NM790-NVMe-SSDs im Homeserver.*

Bei Hardwarefotos ist darauf zu achten, dass keine Seriennummern, MAC-Adressen, QR-Codes, Barcodes oder privaten Aufkleber sichtbar sind.

---

## Ergebnis

Mit dem HP ProDesk 800 G5 Mini steht eine kompakte und flexiblere x86-Plattform für den weiteren Ausbau des Homeserver-Homelabs zur Verfügung.

Die vorhandenen NVMe-SSDs werden als RAID-1-Speicherbasis genutzt. Die ursprüngliche 256-GB-SSD bleibt als externes Laufwerk weiterhin flexibel verwendbar.

---

## Erkenntnisse

* Eine Migration kann sinnvoll sein, auch wenn das bestehende System noch nicht überlastet ist.
* Ein kompakter x86-Business-PC bietet gegenüber einem SBC mehr Flexibilität für spätere Erweiterungen.
* Interne NVMe-Speicheranbindung ist für einen Homeserver deutlich geeigneter als typische microSD- oder USB-basierte Setups.
* Zwei identische SSDs sind eine sinnvolle Grundlage für RAID 1.
* RAID 1 erhöht die Verfügbarkeit, ersetzt aber kein Backup.
* Vorhandene Hardware kann durch gezielte Umverteilung effizient weiterverwendet werden.