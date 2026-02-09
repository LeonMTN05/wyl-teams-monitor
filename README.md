# wyl-teams-monitor
**WatchYourLAN → Microsoft Teams Notifier (Docker Compose)**

[![Docker Compose](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Teams Webhook](https://img.shields.io/badge/Microsoft%20Teams-Incoming%20Webhook-6264A7?logo=microsoftteams&logoColor=white)](https://learn.microsoft.com/)

Ein schlanker Service, der die **WatchYourLAN API** regelmäßig abfragt und bei **Statuswechseln (ONLINE/OFFLINE)** eine Nachricht an einen **Microsoft Teams Incoming Webhook** sendet.

> ✅ Beim ersten Start wird nur initialisiert (State aufbauen) – **keine Benachrichtigungen**, damit es keinen Alert-Sturm gibt.

<details>
  <summary>📸 Screenshot (Beispiel)</summary>

  ![Teams Alert Example](https://i.imgur.com/cRddQ3h.png)
</details>

---

## ✨ Features

- ONLINE/OFFLINE Alerts in Microsoft Teams
- Gerätedetails in der Nachricht: Name, Hersteller, MAC, IP, Zeitstempel, WYL „Last Seen“
- Optional: nur Geräte mit `Known=1` überwachen
- Offline-„Debounce“: erst nach X fehlenden Polls als offline werten
- Persistenter State im Docker-Volume (`/data/state.json`)
- Läuft direkt mit `python:3.12-slim` (installiert `requests` beim Start)

---

## ✅ Schnellstart

```bash
docker compose up -d
docker compose logs -f wyl-teams-monitor
```

## ⚙️ Konfiguration

### Pflicht

- **WYL_API_URL**: WatchYourLAN API Endpoint (z. B. `http://<host>:8840/api/all`)
- **TEAMS_WEBHOOK_URL**: Microsoft Teams Incoming Webhook URL

### Optional

| Variable | Default | Beschreibung |
| :--- | :--- | :--- |
| `TZ` | `Europe/Berlin` | Zeitzone |
| `POLL_INTERVAL` | `15` | Poll-Intervall (Sekunden) |
| `HTTP_TIMEOUT` | `10` | Timeout (Sekunden) für API/Webhook |
| `TRACK_KNOWN_ONLY` | `true` | Nur `Known=1` überwachen |
| `OFFLINE_IF_MISSING_AFTER` | `3` | Offline nach X fehlenden Polls (wenn zuvor online) |
| `DEBUG` | `false` | Mehr Logs |
| `STATE_PATH` | `/data/state.json` | State-Datei |

## 🔐 Empfehlung: .env statt Hardcoding

Erstelle eine `.env` (nicht committen):

```ini
WYL_API_URL=http://<dein-wyl-host>:8840/api/all
TEAMS_WEBHOOK_URL=https://<dein-teams-webhook>
```

Optional kannst du auch Defaults darüber pflegen:

```ini
TZ=Europe/Berlin
POLL_INTERVAL=15
HTTP_TIMEOUT=10
TRACK_KNOWN_ONLY=true
OFFLINE_IF_MISSING_AFTER=3
DEBUG=false
STATE_PATH=/data/state.json
```

## 🧠 Wie es funktioniert (kurz)

WatchYourLAN liefert pro Gerät u. a.:
- `Now` (1=online, 0=offline)
- `Known` (für Filter `TRACK_KNOWN_ONLY`)

Der Monitor speichert den letzten Status pro Gerät in `STATE_PATH`.
Nur wenn sich `Now` ändert, wird eine Teams-Nachricht gesendet.

Wenn ein Gerät temporär aus der API-Liste verschwindet, wird es erst nach `OFFLINE_IF_MISSING_AFTER` verpassten Polls als offline gemeldet.

## 💾 Persistenz (State)

Der Zustand wird unter `STATE_PATH` gespeichert (Standard: `/data/state.json`) und via Volume `wyl-teams-monitor-data` persistiert.

State zurücksetzen (löscht gespeicherte Status!):

```bash
docker compose down -v
docker compose up -d
```

## 🛠️ Troubleshooting

**Keine Benachrichtigungen nach dem Start?**
Normal: erster Durchlauf = Initialisierung ohne Alerts. Danach nur bei Änderungen.

**API liefert nicht das erwartete JSON?**
`WYL_API_URL` muss ein JSON-Array (Liste) liefern:

```bash
curl -sS "$WYL_API_URL" | head
```

**Teams Webhook Fehler (HTTP 4xx/5xx)?**
Webhook URL prüfen, Connector aktiv, Netzwerk/Proxy/Firewall, ggf. `HTTP_TIMEOUT` erhöhen.

## 📎 Drittanbieter / Marken / Copyright

WatchYourLAN ist ein separates Open-Source-Projekt (Urheberrecht & Lizenz liegen beim WatchYourLAN-Projekt).
Repository: https://github.com/aceberg/WatchYourLAN

Microsoft und Microsoft Teams sind Marken der Microsoft-Unternehmensgruppe.
Dieses Projekt ist keine offizielle Microsoft-Integration.

Dieses Repository enthält keinen Code aus WatchYourLAN und nutzt lediglich dessen HTTP-API sowie Microsoft Teams Incoming Webhooks.
