# ESPHome Configuration Repository

## Projektbeschreibung

Dieses Repository enthält ESPHome-Konfigurationsdateien für verschiedene Smart-Home-Geräte. Die Konfigurationen nutzen ein Package-System mit einer gemeinsamen Basis-Konfiguration.

## Dateistruktur

```
.
├── common.yaml              # Gemeinsame Basis-Konfiguration (WiFi, API, OTA, MQTT, etc.)
├── secrets.yaml.example     # Vorlage für Secrets (wird ins Repository committed)
├── secrets.yaml             # Echte Secrets (wird NICHT committed, siehe .gitignore)
├── power-meter.yaml         # Stromzähler-Konfiguration (SML)
├── gas-meter.yaml           # Gaszähler-Konfiguration (Reed-Kontakt)
├── smartsolar.yaml          # Victron SmartSolar MPPT Laderegler
├── smartshunt.yaml          # Victron SmartShunt Batteriemonitor
└── .gitignore              # Schützt secrets.yaml vor versehentlichem Commit
```

## Sicherheitskonzept

### Secrets Management

**KRITISCH:** Keine Passwörter oder sensible Daten dürfen in YAML-Dateien committed werden!

- Alle sensiblen Werte werden über `!secret` Platzhalter referenziert
- `secrets.yaml` ist in `.gitignore` und wird NIEMALS committed
- `secrets.yaml.example` dient als Vorlage mit Platzhaltern

### Benötigte Secrets

In `secrets.yaml` müssen folgende Werte definiert sein:

- `wifi_ssid` - WLAN-Name
- `wifi_password` - WLAN-Passwort
- `ap_password` - Fallback-AP Passwort
- `encryption_key` - API-Verschlüsselung (Base64, 32 Bytes)
- `ota_password` - OTA-Update Passwort
- `mqtt_broker` - MQTT-Broker IP/Hostname

## Workflow

### Neue Gerätekonfiguration hinzufügen

1. Neue YAML-Datei erstellen (z.B. `neues-geraet.yaml`)
2. Common-Package einbinden:
   ```yaml
   packages:
     common:
       url: https://github.com/legacycode/ESPHome-Config.git
       file: common.yaml
       ref: main
       refresh: 0s  # Immer aktuellste Version beim Kompilieren laden
   ```
3. Konfiguration validieren: `esphome config neues-geraet.yaml`
4. Bei erfolgreichem Test committen

**Hinweis:** `refresh: 0s` lädt die Remote-Packages bei jedem Kompilieren neu. Die ESP-Geräte aktualisieren sich **nicht automatisch** - du musst nach Änderungen manuell neu kompilieren und flashen.

### Vor jedem Commit

**PFLICHT:** Konfiguration validieren:
```bash
esphome config <datei>.yaml
```

Die Konfiguration MUSS erfolgreich validiert werden, bevor sie committed wird.

### Git Workflow

```bash
# Änderungen validieren
esphome config power-meter.yaml

# Bei Erfolg committen
git add <dateien>
git commit -m "Beschreibung"
git push
```

**WICHTIG:** Keine Co-Authorship von Claude in Commits.

## Common Configuration

Die `common.yaml` enthält:

- ESPHome Core-Konfiguration
- Logger (Level: INFO)
- WiFi mit Fallback-AP
- API mit Verschlüsselung
- Zeit-Synchronisation (Home Assistant)
- OTA-Updates mit Passwort
- MQTT-Integration
- Web-Server (Port 80)
- Captive Portal (Fallback)
- Restart-Button (diagnostic)

### Diagnostic-Sensoren (automatisch für alle Geräte)

**Sensoren:**
- Betriebszeit (Sekunden, update_interval: 60s)

**Text-Sensoren:**
- ESPHome Version

**Binary-Sensoren:**
- Status (Verbindungsstatus)

**Wichtig:** Nach Hinzufügen neuer Sensoren zur `common.yaml` müssen Geräte in Home Assistant möglicherweise gelöscht und neu hinzugefügt werden, damit die Discovery korrekt funktioniert. Die Sensoren erscheinen dann in der Diagnostic-Sektion.

**Branch `smartsolar-reduce-sensors`:** Experimenteller Branch mit minimal reduzierten Sensoren (ohne Debug- und WiFi-Info-Sensoren) zur Reduzierung der ESP8266-Last.

## Geräte

### power-meter.yaml

SML-Stromzähler Ausleser auf Wemos D1 Mini:

- **Board:** ESP8266 (d1_mini)
- **UART:** GPIO1 (TX), GPIO3 (RX), 9600 Baud
- **Sensoren:**
  - Bezug gesamt (OBIS 1-0:1.8.0)
  - Einspeisung gesamt (OBIS 1-0:2.8.0)
  - Aktuelle Wirkleistung (OBIS 1-0:16.7.0)
  - Stromzähler-ID (OBIS 1-0:96.1.0)

### gas-meter.yaml

Gaszähler-Ausleser mit Reed-Kontakt auf Wemos D1 Mini:

- **Board:** ESP8266 (d1_mini)
- **Reed-Kontakt:** GPIO4 (mit Pullup)
- **Remote Packages:** github://legacycode/ESPHome-Gas-Meter
- **Konfiguration:**
  - `pulses_per_cubic_meter: "100"` - Impulse pro m³
  - `initial_meter_offset: "0"` - Initialer Zählerstand-Offset
- **Sensoren:**
  - Durchflussrate (m³/h)
  - Gesamt (m³)
  - Gesamtimpulse
  - Zählerstand mit Offset (m³)
  - WiFi-Signal, Betriebszeit
- **Funktionen:**
  - LED-Blinken bei Impuls (3s)
  - Impulszähler zurücksetzen (Button)
  - Zählerstand-Offset konfigurierbar (Number)
  - Persistente Speicherung der Impulse

**Wichtig:** Die Sensor-Konfiguration wird aus dem externen Repository `github://legacycode/ESPHome-Gas-Meter` geladen. Änderungen an Sensoren, Pins oder Funktionen müssen dort vorgenommen werden, nicht in dieser `gas-meter.yaml`. Diese Datei enthält nur die gerätespezifischen Substitutions (devicename, pulses_per_cubic_meter, etc.).

### smartsolar.yaml

Victron SmartSolar MPPT Laderegler auf Wemos D1 Mini:

- **Board:** ESP8266 (d1_mini)
- **UART:** RX=D7 (GPIO13), 19200 Baud (nur RX, da VE.Direct Text Protocol read-only ist)
- **Remote On/Off:** D6 (GPIO12) steuert Victron RX-Pin für ferngesteuertes Ein-/Ausschalten
  - HIGH (3.3V) = Gerät EIN
  - LOW (GND) = Gerät AUS
  - `restore_mode: ALWAYS_ON` - Gerät bleibt nach Neustart eingeschaltet
- **Externe Komponente:** github://KinDR007/VictronMPPT-ESPHOME@main
- **Beispielkonfiguration:** https://github.com/KinDR007/VictronMPPT-ESPHOME/blob/main/smartsolar-mppt-esp8266-example.yaml
- **Sensoren (14):**
  - Panel Spannung/Leistung (mit device_class: voltage/power, state_class: measurement)
  - Batterie Spannung/Strom (mit device_class: voltage/current, state_class: measurement)
  - Ertrag (Gesamt, Heute, Gestern) in kWh (mit device_class: energy, state_class: total_increasing)
  - Max Leistung (Heute, Gestern) (mit device_class: power, state_class: measurement)
  - Tag Nummer, Lademodus ID, Fehlercode, Tracking Modus ID
  - Last Strom (mit device_class: current, state_class: measurement)
- **Text-Sensoren (6):**
  - Lademodus, Tracking Modus, Fehler
  - Firmware Version, Gerätetyp, Seriennummer
- **Binary-Sensoren (2):**
  - Last Status, Relais Status
- **Switch (1):**
  - Remote On/Off (steuert Gerät über Victron RX-Pin)

### smartshunt.yaml

Victron SmartShunt Batteriemonitor auf Wemos D1 Mini:

- **Board:** ESP8266 (d1_mini)
- **UART:** RX=D7 (GPIO13), 19200 Baud (nur RX, da VE.Direct Text Protocol read-only ist)
- **Externe Komponente:** github://KinDR007/VictronMPPT-ESPHOME@main
- **Beispielkonfiguration:** https://github.com/KinDR007/VictronMPPT-ESPHOME/blob/main/smartshunt-esp8266-example.yaml
- **Sensoren (~15, reduziert):**
  - Batterie Spannung/Strom/Ladezustand (mit device_class: voltage/current/battery, state_class: measurement)
  - Hilfsbatterie Spannung (mit device_class: voltage, state_class: measurement)
  - Momentanleistung (mit device_class: power, state_class: measurement)
  - Verbrauchte Ah, Restlaufzeit
  - Entladungstiefen (Tiefste, Letzte, Durchschnittlich)
  - Min/Max Batterie Spannung (mit device_class: voltage, state_class: measurement)
  - Min/Max Hilfsbatterie Spannung (mit device_class: voltage, state_class: measurement)
  - Energie geladen/entladen in kWh (mit device_class: energy, state_class: total_increasing)
  - DC Monitor Modus ID
  - Letzte Vollladung, Kumulierte Ah entnommen
- **Text-Sensoren (3):**
  - Modellbeschreibung, Firmware Version, DC Monitor Modus
- **Energy Dashboard:**
  - `amount_of_charged_energy` → Energie IN die Batterie (kWh)
  - `amount_of_discharged_energy` → Energie AUS der Batterie (kWh)
- **Hinweis:** Kein Remote On/Off Switch (im Gegensatz zum SmartSolar)

## Hinweise für Claude

- Vor Commits IMMER `esphome config` ausführen
- **CLAUDE.md vor jedem Push aktualisieren!**
- Keine Secrets in YAML-Dateien hardcoden
- Für Validierung Test-Secrets verwenden (secrets.yaml wird nicht committed)
- Keine Co-Authorship in Git-Commits
- Bestehende Konventionen in common.yaml beachten
