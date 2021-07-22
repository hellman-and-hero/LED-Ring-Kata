## Die Ausgangslage

Die Legacy Anwendung [https://github.com/hellman-and-hero/musa-client](https://github.com/hellman-and-hero/musa-client) soll erweitert werden. Die Werte der zwei Slider welche auf dem Tab "Stock P/A" zu finden sind sollen zukünftig bei Veränderungen auch auf LED-Ringen visualisiert werden. Dazu muss die Methode [https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockPanel.java#L65](https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockPanel.java#L65) angepasst werden. Die StockPanel-Klasse wird vom Framework instanziiert. Daher darf die Signatur des Konstruktors nicht verändert werden, ebenso nicht der Inhalt des Interfaces welche im Konstruktor übergeben wird.

Pro Schieberegler auf dem Panel soll der jeweils aktuelle Wert mit je einem LED-Ring dargestellt werden. Diese Hardware ist von einem Fremdhersteller zugekauft.

- In der Hardware sind 32 LEDs in Form von zwei Ringen je 16 LEDs verbaut. Diese 32 LEDs sind logisch hintereinander angeordnet, d.h. die LEDs 1-16 befinden sich im ersten, die LEDs 17-32 im zweiten Ring <sup>[1](#myfootnote1)</sup>
- Für jede einzelne LED muss die darzustellende Farbe an die Hardware gesendet werden, sollen also die ersten drei LEDs "geschaltet" werden, sind drei Nachrichten zu senden <sup>[2](#myfootnote1)</sup> <sup>[3](#myfootnote1)</sup>
- Die Werte der Schieberegler gehen von 0-100. Je nach Wert sollen entsprechend viele LEDs leuchten. Beispielsweise soll bei einem Ring mit 16 LEDs und einem Wert größer 0 die erste LED leuchten, bei einem Wert größer 25 die ersten vier, bei einem Wert größer 50 die ersten acht. Bei einem Wert 0 sollen alle LEDs aus sein.
- Neben der Hardware mit 16 LEDs pro Ring existieren noch Varianten mit 8 bzw. 12 LEDs pro Ring. Daher muss die Anzahl der LEDs konfiguriert werden können, damit alle drei Varianten genutzt werden können.
- Den Namen des MQTT-Hosts, dessen Port, sowie die Anzahl der LEDs für Ring 1 und Ring 2 liegen im [StockContext](https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockContext.java). welches das [StockPanel im Konstruktor](https://github.com/hellman-and-hero/musa-client/blob/5f1060012d5b2427bfd9ad4bf7e15348fd562949/src/main/java/musa/stock/StockPanel.java#L39) übergeben bekommt.
- Da Du die echte Hardware nicht zur Verfügung hast, kannst Du [die Softwaresimulation der Hardware](https://github.com/hellman-and-hero/tdd-demy-hardware-sim) nutzen. Die Simulation stellen wir im Rahmen der Bearbeitung der Kata der echten Hardware gleich (was wir normalerweise nicht tun dürfen, da sich die Simulation anders verhalten kann/könnte als die echte Hardware).

<a name="myfootnote1">1</a>: Eine bildliche Darstellung hiervon befindet sich auf Seite 11 des Foliensatzes von [TDD demystified](https://www.xpdays.de/2018/downloads/174-tdd-demystified/tdd_demystified.pdf)

<a name="myfootnote2">2</a>: Technisch ist je LED eine MQTT Nachricht an den MQTT-Broker zu senden, mit welchem auch die Hardware verbunden ist. Die Nachricht muss das Topic "some/led/<led-nummer>/rgb" und als payload die Farbe als Hexwert ("#RRGGBB") haben.

<a name="myfootnote3">3</a>: Falls Du Dir das Versenden von MQTT-Nachrichten nicht selbst erarbeiten möchtest (ist nicht Ziel dieser Kata), kannst Du auf dem ["simplified" branch](https://github.com/hellman-and-hero/musa-client/tree/simplified) starten und dort die Klasse [MqttSender](https://github.com/hellman-and-hero/musa-client/blob/simplified/src/main/java/rgbledring/MqttSender.java) nutzen. Hast Du keinen MQTT Broker, kannst Du Dir entweder einfach mosquitto installieren, [mosquitto via docker](https://hub.docker.com/_/eclipse-mosquitto) (_docker run -it -p 1883:1883 -p 9001:9001 eclipse-mosquitto:1.6_) oder einen [öffentlichen MQTT-Broker](https://github.com/mqtt/mqtt.org/wiki/public_brokers) nutzen.

### Sprint Backlog Sprint #1

1. Der Wert des ersten Schiebereglers soll auf dem ersten Ring visualisiert werden. Im Sprint #1 sind Farben noch nicht wichtig, es reicht aus, wenn LEDs an- (#FFFFFF) und aus- (#000000) geschaltet werden können!
2. Richtung der LEDs
 Die Richtung kann je Ring konfiguriert werden (im oder gegen den Uhrzeigersinn)
3. Zweiter Ring
 Analog zum ersten Ring soll nun der Wert des zweiten Schiebereglers auf dem zweiten Ring visualisiert werden. Auch hier, erst einmal ohne Farben

**Du hast die Anforderungen umgesetzt? Klasse! Dann weiter zur nächsten Seite!**

### Sprint Backlog Sprint #2

1. Die LEDs der Ringe sollen farbig leuchten (erstes Drittel grün, mittleres Drittel gelb, letztes Drittel rot). Soll für Ringe mit 8 (3/2/3), 12 (4/4/4) und 16 (5/6/5) LEDs funktionieren. Ringe anderer Größen müssen nicht berücksichtigt werden.
2. Overload
 Der Ring soll von 0-90 so befüllt werden wie zuvor von 0-100, d.h. 90 bedeutet: Alle LEDs an (ehemals 100), 45 bedeutet: Die erste Hälfte der LEDs an (ehemals 50). Bei mehr als 90 leuchten **alle** LEDs **rot**. Der Schwellwert (die 90) soll variabel sein (kann also z.B. auch 80, 95 oder jeder beliebige andere Wert sein)

**Du hast die Anforderung umgesetzt? Klasse! Dann weiter zur nächsten Seite!**

### Sprint Backlog Sprint #3

1. Peak-Hold
 Bei Abfallen des Levels soll die LED welche den Maximalwert signalisierte, noch eine Sekunde lang nachleuchten
2. Overhead
 Damit die Hardware auch an der Decke montiert werden kann soll Start/Ende der Ringe konfigurierbar sein. Auch eine seitliche Montage an der Wand soll möglich sein. So wären im ersten Fall (Decke) bei Ringen mit 8 LEDs die Reihenfolge der zu schaltenden LEDs 5,6,7,8,1,2,3,4 und im zweiten (Wand rechts 7,8,1,2,3,4,5,6 bzw. Wand links 3,4,5,6,7,8,1,2). Diese Einstellung gilt für die gesamte Hardware und damit für alle Ringe gleichermasen. 
 
**Du hast die Anforderung umgesetzt? Klasse! Dann weiter zur nächsten Seite!**

### Sprint Backlog Sprint #4

1. Die einzelnen Features sollen je Ring ein-/ausschaltbar bzw. konfigurierbar sein. Je Ring kann also konfiguriert werden:

- Anzahl LEDs
- Richtung im oder gegen den Uhrzeigersinn
- Overload an/aus (inkl. individuellem Schwellwert)
- Peak-Hold an/aus
- Konfiguration der Position der ersten LED der Ringe

**Du hast die Anforderung umgesetzt? Herzlichen Glückwunsch! Der Product Owner hat keine weiteren Anforderungen. Auf der nächsten Seite findest Du noch ein paar Anmerkungen**

## Anmerkungen
- Bist Du über den Index der LEDs gestolpert (Beschreibung spricht von LEDs 1-n, die Hardware geht aber von einer Null-basierten Nummerierung aus)? Dies fällt nur durch Ausprobieren oder Integrationstests mit der echten Hardware (bzw. unserer simulierten Hardware) auf.
- Welche Auswirkungen auf Deinen Produktiv- und Testcode hätte die Anforderung, dass eine neue Hardwareversion unterstützt werden soll, welche nicht per MQTT sondern über den seriellen Port angesteuert werden soll (die bestehende Unterstützung für MQTT ist nach einer Übergangszeit nicht mehr notwendig und kann entfernt werden)?
- Hast Du einen öffentlichen MQTT Broker in Deinen Tests benutzt? Einen in Deinem Umfeld existierenden? Was hat dies für Konsequenzen für Deine Tests? Was wären Alternativen?
