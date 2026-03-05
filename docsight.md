---
title: Docsight
description: A guide to deploying Docsight
published: true
date: 2026-03-05T15:12:42.220Z
tags: 
editor: markdown
dateCreated: 2026-03-05T15:12:42.220Z
---

# <img src="/docsight.png" class="tab-icon"> What is DOCSight?

**DOCSight** is a self-hosted DOCSIS cable modem monitoring dashboard that tracks your cable internet connection 24/7, providing health assessments, trend charts, and hard evidence for ISP complaints. It features automatic signal quality evaluation, interactive charts with DOCSIS reference zones, Speedtest Tracker integration, complaint letter generation, and optional Home Assistant integration via MQTT auto-discovery.

> DOCSight is designed exclusively for **cable internet (DOCSIS/coax)** connections. It will not work with DSL or fiber.
{.is-danger}




# <img src="/docker.png" class="tab-icon"> 1 · Deploy DOCSight

```yaml
services:
  docsight:
    image: ghcr.io/itsdnns/docsight:latest
    container_name: docsight
    environment:
      - MODEM_URL=http://192.168.178.1
      - MODEM_USER=admin
      - MODEM_PASSWORD=yourpassword
      - POLL_INTERVAL=900
      - WEB_PORT=8765
      - HISTORY_DAYS=0
      # - ADMIN_PASSWORD=changeme
      # - MQTT_HOST=192.168.1.x
      # - MQTT_PORT=1883
      # - MQTT_USER=
      # - MQTT_PASSWORD=
      # - MQTT_TOPIC_PREFIX=docsight
      # - BQM_URL=
      # - SPEEDTEST_TRACKER_URL=
      # - SPEEDTEST_TRACKER_TOKEN=
    restart: unless-stopped
    ports:
      - "8765:8765"
    volumes:
      - /mnt/tank/configs/docsight:/data
```

1. Update `MODEM_URL`, `MODEM_USER`, and `MODEM_PASSWORD` to match your cable modem / router login.
1. On first launch, the **Setup Wizard** will guide you through connection testing and configuration.

> 
> Environment variables are **optional** — DOCSight includes a browser-based Setup Wizard that writes all settings to `config.json` inside the `/data` volume. You can skip all env vars and configure everything through the web UI.
{.is-info}

# 2 · Configuration

## 2.1 Environment Variables

Configuration is stored in `config.json` inside the Docker volume and persists across restarts. Environment variables override `config.json` values.

| Variable | Default | Description |
|---|---|---|
| `MODEM_URL` | `http://192.168.178.1` | Modem / router URL |
| `MODEM_USER` | — | Modem username |
| `MODEM_PASSWORD` | — | Modem password |
| `MQTT_HOST` | — | MQTT broker host (optional) |
| `MQTT_PORT` | `1883` | MQTT broker port |
| `MQTT_USER` | — | MQTT username (optional) |
| `MQTT_PASSWORD` | — | MQTT password (optional) |
| `MQTT_TOPIC_PREFIX` | `docsight` | MQTT topic prefix |
| `POLL_INTERVAL` | `900` | Polling interval in seconds |
| `WEB_PORT` | `8765` | Web UI port |
| `HISTORY_DAYS` | `0` | Snapshot retention in days (`0` = unlimited) |
| `ADMIN_PASSWORD` | — | Web UI password (optional) |
| `BQM_URL` | — | ThinkBroadband BQM share URL (optional) |
| `SPEEDTEST_TRACKER_URL` | — | Speedtest Tracker URL (optional) |
| `SPEEDTEST_TRACKER_TOKEN` | — | Speedtest Tracker API token (optional) |
{.dense}

## 2.2 Web UI Settings

Access `/settings` at any time to change configuration, set an admin password, or toggle light/dark mode. The Setup Wizard runs automatically on the first start.

## 2.3 Home Assistant Integration (Optional)

DOCSight can publish all channel data to Home Assistant via **MQTT Auto-Discovery**. To enable:

1. Set `MQTT_HOST` to your MQTT broker address (e.g., Mosquitto).
2. Optionally configure `MQTT_USER`, `MQTT_PASSWORD`, and `MQTT_TOPIC_PREFIX`.
3. Restart the container — DOCSight will automatically create HA entities.

Summary sensors created include overall health status, downstream/upstream channel counts, power min/max/avg, SNR min/avg, and correctable/uncorrectable error totals.

## 2.4 Speedtest Tracker Integration (Optional)

DOCSight can pull speed test results from a self-hosted [Speedtest Tracker](https://github.com/alexjustesen/speedtest-tracker) instance. To enable:

1. Set `SPEEDTEST_TRACKER_URL` to your Speedtest Tracker instance URL.
2. Set `SPEEDTEST_TRACKER_TOKEN` to your API token.
3. The DOCSight dashboard will show speed test charts, sortable history, and anomaly highlighting.



