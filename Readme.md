![Preview](https://github.com/Stone-Time/HomeAssistant-BambuLab-DashBoard/blob/main/screenshot.jpg?raw=true)



============================ Erforderliche HACs Komponenten
- Mushroom
- card-mod
- Stack In Card
- Bambu Lab
- Google Dark Theme

============================ Drucker importieren in BambuLab Geräte
[] Bambu Cloud Configuration, nicht LAN!
[] Europe

[] Verbinden Sie sich direkt mit dem Drucker statt über die Bambu Cloud <--- aktivieren


============================ Drucker importieren in BambuLab Geräte
-> Einstellungen -> Geräte & Dienste -> Integration 
-> Generiesche Kamera anlegen

Standbild URL: (leer lassen)
Stream Quell URL: rtsps://<printer-ip>/streaming/live/1
RTSP Protokol: TCP
Authentifizierung: basic
Benutzername: bblp
Passwort: <lan-access-code>
Bildfrequenz: 30
SSL-Zertifikat überprüfen: deaktivieren


============================ Inhalte des www Ordners übertragen
Entweder über SambaShare oder File editor einzeln 
www Ordner sollte existieren, die anderen müssen angelegt werden


============================ Daten ermitteln um diese nachfolgend und im Dashboard zu ersetzen

================= Drucker             BEISPIELE!
<BAMBULABPRINTER_ID> 				- x1c_00m00a000000000
<BAMBULABPRINTER_AMS_ID> 			- x1c_00m00a000000000_ams

================= Powerschalter
<POWER_SWITCH_ID>					- switch.powerprinterrack1_2
<POWER_POWERMETER_ID>				- sensor.powerprinterrack1_2_power
<POWER_VOLTAGEMETER_ID>				- sensor.powerprinterrack1_2_voltage
<POWER_CURRENTMETER_ID>				- sensor.powerprinterrack1_2_current
<POWER_DAYENERGYMETER_ID>			- sensor.powerprinterrack1_2_total_daily_energy
<POWER_TOTALENERGYMETER_ID>			- sensor.powerprinterrack1_2_energy

================= Kamera
<CAM_ID>							- camera.10_10_5_91

============================ Zähler als Helfer anlegen 
-> Einstellungen -> Geräte & Dienste -> Helfer
-> Helfer erstellen -> Zähler


name: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_carbonrods
icon mdi:wrench-clock
Minimaler Wert: 0
Anfangswert: 0
Schrittgröße: 1

name: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_yzrods
icon mdi:wrench-clock
Minimaler Wert: 0
Anfangswert: 0
Schrittgröße: 1

name: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_zspindle
icon mdi:wrench-clock
Minimaler Wert: 0
Anfangswert: 0
Schrittgröße: 1

name: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_maindevice
icon mdi:wrench-clock
Minimaler Wert: 0
Anfangswert: 0
Schrittgröße: 1



============================ Automatisierung anlegen
-> Einstellungen -> Automatisierungen & Szenen
-> Automatisierung erstellen -> Neue Automatisierung erstellen
-> Rechts oben, 3 Punkte -> Als YAML bearbeiten


alias: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_count
description: ""
trigger:
  - platform: state
    entity_id:
      - sensor.<BAMBULABPRINTER_ID>_gesamtnutzung
    alias: printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_count
condition:
  - condition: template
    value_template: |
      {% set t_new = trigger.to_state.state|default(0)|int %}
      {% set t_from = trigger.from_state.state|default(0)|int %}
      {{ t_new >= t_from }}
action:
  - service: counter.increment
    metadata: {}
    data: {}
    target:
      entity_id:
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_carbonrods
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_maindevice
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_yzrods
        - counter.printer_bambulab_<BAMBULABPRINTER_ID>_maintenance_zspindle
mode: single



-> Speichern -> Umbennen -> 

============================ Seite anlegen und alle <IDs> in yaml ersetzen

YAML Template der Seite je nachdem was beim Bambu in den Entitäten stehen

Heisst der "Aktiver Slot" als Entitäts-ID "active_tray" am ende -> en_yaml
Heisst der "Aktiver Slot" als Entitäts-ID "aktiver_slot" am ende -> de_yaml