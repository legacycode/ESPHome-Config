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
       refresh: 1d
   ```
3. Konfiguration validieren: `esphome config neues-geraet.yaml`
4. Bei erfolgreichem Test committen

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
- Restart-Button

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

## Hinweise für Claude

- Vor Commits IMMER `esphome config` ausführen
- Keine Secrets in YAML-Dateien hardcoden
- Für Validierung Test-Secrets verwenden (secrets.yaml wird nicht committed)
- Keine Co-Authorship in Git-Commits
- Bestehende Konventionen in common.yaml beachten
