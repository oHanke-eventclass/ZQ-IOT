# Anleitung Help-Desk

## Geräte/Materialien:
- Raspberry Pi 5
- Breadboard & Jumper-Kabel
- 3x LEDs mit Vorwiderständen
	- Grün = System bereit / Alles Okay
	- Rot = SOS ausgelöst
	- Gelb = SOS vom Admin angenommen
- 1x Taster
- Monitor, Tastatur & Maus
- Laptop
- WLAN-/Netzwerkverbindung

## Software:
Raspberry PI
- Raspberry Pi OS
- Python3
	- gpiozero
	- requests

Laptop / Server
- Docker
- Node-Red

## Durchführung:
- Raspberry Pi einrichten
	- Raspberry Pi mit Monitor, Tastatur und Maus verbinden.
	- Raspberry Pi OS installieren.
	- Terminal öffnen, GPIO-Pinbelegung anzeigen:
			- Befehl: pinout
	- Python3 installieren (falls noch nicht vorinstalliert) zum Programmieren der Anwendung

- Breadboard vorbereiten und anschließen
	- LEDs mit Vorwiderständen einzeln aufstecken
	- Taster aufstecken
	- Ground von GPIO Pin als Quelle für Ground auf dem Breadboard verwenden mithilfe von Jumper-Kabel
	- LEDs und Taster je mit Raspberry Pi verbinden an GPIO Pins mit Jumper-Kabel (pinout Ausgabe beachten)


- App.py auf Pi erstellt
	- Testen ob GPIO Pins ansprechbar sind (LEDs zum leuchten bringen, Taster input registrierbar etc.)
	- mit gpiozero werden LEDs automatisch als Ausgänge und der Taster als Eingang behandelt
	- beim Start soll zunächst gelten:
		- Grün - AN
		- Gelb - AUS
		- Rot - AUS
	- wird der Taster gedrücke:
		- Grün - AUS
		- Gelb - AUS
		- Rot - AN

- Node-RED & Docker auf Laptop einrichten
	- Docker installieren
	- Node-RED installieren
	- Node-RED über Docker starten
		- Volume erstellen
			- docker volume create --name node_red_data
		- in Docker-Container ausführen mit Port als Zugang freigeben
			- docker run -it -p 1880:1880 -v node_red_data:/data --name mynodered nodered/node-red --dns 8.8.8.8
				- damit wird Port 1880 des Containers auf Port 1880 des Laptops veröffentlicht und ist anschließlich über http://127.0.0.1:1800 lokal erreichbar
				- von anderen Geräten über die tatsächliche IP des Node-Red-Laptops, z.B. http://192.168.178.50:1880

- Verbindung zwischen Raspberry Pi und Node-RED
	- vom Raspberry Pi kann zunächst im Terminal getestet werden: ping IP-des-Node-RED-Laptops
	- anschließend curl http://IP-des-Node-RED-Laptops:1880

- Node-RED SOS empfangen
	- in Node-RED einen neuen Flow erstellen
		- [HTTP] POST /sos -> [Function] SOS setzen -> [HTTP Response]

- Raspberry Pi sendet das SOS
	- im Python-Programm kann beim Tastendruck eine HTTP-Anfrage an Node-RED geschickt werden, dafür wird hier requests verwendet: import requests
	- Node-RED-Adresse angeben: SERVER = "http:/IP-des-Node-RED-Laptops:1880"
	- beim Tastendruck: requests.post(SERVER + "/sos")

- Admin-Webseite erstellen
	- Node-RED stellt außerdem die Help-Desk-Webseite bereit, dafür einen zweiten Flow erstellen
		- [HTTP] GET /helpdesk -> [Function] SOS auslesen -> [Template] html/css -> [HTTP Response]
		- Seite ist dann beispielsweise erreichbar unter http://IP-DES-NODE-RED-LAPTOPS:1880/helpdesk
		- das html kann sich auch automatisch aktualisieren um den aktuellen Status anzuzeigen
			- z.B. refresh aller 10 Sekunden <meta http-equiv="refresh" content="10">

- SOS vom Admin quittieren
	- als Erweiterung kann der Admin auf der Webseite einen Button erhalten
		- [HTTP] POST /reset -> [Function] SOS zurücksetzen -> [HTTP Response]
		- daraus wird aus sos = true wieder sos = false

- Status zurück zum Raspberry Pi
	- der Raspberry Pi kann zusätzlich regelmäßig durch eine Schleife einen Status-Endpunkt von Node-RED abfragen
	
- in der settings.js Datei von Node-Red kann man unter adminAuth auch einen user und bcrypt hashed Passwort setzen, um den Zugriff auf /helpdesk etwas abzusichern

