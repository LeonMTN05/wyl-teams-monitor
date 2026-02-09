# wyl-teams-monitor  
**WatchYourLAN → Microsoft Teams Notifier (Docker Compose)**

[![Docker Compose](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Status](https://img.shields.io/badge/Status-Ready-brightgreen)](#)

Ein kleiner, selbstenthaltener Docker-Compose-Service, der die **WatchYourLAN API** in einem festen Intervall abfragt und bei **Statuswechseln (ONLINE/OFFLINE)** eine Benachrichtigung an einen **Microsoft Teams Incoming Webhook** sendet.

> 💡 Beim ersten Start wird nur initialisiert (State aufbauen) – **keine Benachrichtigungen**, damit es keinen „Alert-Sturm“ gibt.

---

## Inhalt
- [Features](#features)
- [Voraussetzungen](#voraussetzungen)
- [Quickstart](#quickstart)
- [Konfiguration](#konfiguration)
- [Wie es funktioniert](#wie-es-funktioniert)
- [Persistenz (State)](#persistenz-state)
- [Security-Hinweise](#security-hinweise)
- [Troubleshooting](#troubleshooting)
- [Rechtliches / Hinweise zu Drittprodukten](#rechtliches--hinweise-zu-drittprodukten)

---

## Features

- ✅ Polling der WatchYourLAN API (`/api/all`)
- ✅ Teams Nachricht bei **Statuswechsel** (online/offline)
- ✅ Enthält Geräteinfos: Hostname, Hersteller, MAC, IP, WYL Last Seen, Zeitstempel
- ✅ Optional: nur Geräte mit `Known=1` überwachen
- ✅ „Offline-Buffer“: Gerät wird erst nach X fehlenden Polls als offline gemeldet
- ✅ Persistenter Zustand im Docker-Volume (`/data/state.json`)
- ✅ Läuft ohne eigenes Dockerfile (installiert `requests` beim Containerstart)

---

## Voraussetzungen

- Docker + Docker Compose (Compose V2 empfohlen)
- Erreichbare WatchYourLAN-Instanz inkl. API (z. B. `http://<host>:8840/api/all`)
- Microsoft Teams **Incoming Webhook** URL (Connector)

---

## Quickstart

1) `docker-compose.yml` bereitstellen (oder anpassen).  
2) Starten:

```bash
docker compose up -d
