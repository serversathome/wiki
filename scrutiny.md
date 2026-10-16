---
title: Scrutiny
description: A guide for deploying Scrutiny on TrueNAS and Docker
published: true
date: 2026-10-16T08:54:21.545Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:26.409Z
---

# <img src="/scrutiny.png" class="tab-icon"> What is Scrutiny?

**Scrutiny** is a hard drive health dashboard that collects S.M.A.R.T metrics from every disk in your system, tracks them over time, and warns you *before* a drive fails — instead of after. It wraps `smartctl` in a web UI, stores history in InfluxDB, and replaces the manufacturer's often-useless thresholds with real-world failure rates.

This page covers the **Starosdev fork** (`ghcr.io/starosdev/scrutiny`), which is the actively maintained continuation of the original AnalogJ project. It adds ZFS pool monitoring, Prometheus metrics, Home Assistant MQTT discovery, performance benchmarking, workload insights, API authentication, and a proper mobile UI.

| | Original (AnalogJ) | Starosdev Fork |
|---|---|---|
| Latest version | v0.8.1 (Apr 2024) | Actively released |
| Frontend | Angular 13 | Angular 21 |
| Status | Minimal updates | Actively maintained |
| Image | `ghcr.io/analogj/scrutiny` | `ghcr.io/starosdev/scrutiny` |


> **Migrating from AnalogJ?** Just swap the image name. Your existing SQLite database, InfluxDB data, `scrutiny.yaml` and `collector.yaml` are all fully compatible — no changes needed.
{.is-success}

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/starosdev/scrutiny"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# 1 · Deploy Scrutiny
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker (Omnibus)

The omnibus image contains the web UI, the API, InfluxDB, and the collector in a single container. This is what you want for a single-box homelab.
```yaml
services:
  scrutiny:
    image: ghcr.io/starosdev/scrutiny:latest-omnibus
    container_name: scrutiny
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "8086:8086"
    cap_add:
      - SYS_RAWIO
      - SYS_ADMIN
    environment:
      - COLLECTOR_HOST_ID=truenas          # REQUIRED for ZFS pool status, see below
      - TZ=America/New_York                # cron schedules and report times use this
      - COLLECTOR_CRON_SCHEDULE=0 0 * * *
      - COLLECTOR_RUN_STARTUP=true
      - COLLECTOR_ZFS_CRON_SCHEDULE=*/15 * * * *
      - COLLECTOR_ZFS_RUN_STARTUP=true
    volumes:
      - /run/udev:/run/udev:ro
      - /mnt/tank/configs/scrutiny/config:/opt/scrutiny/config
      - /mnt/tank/configs/scrutiny/influxdb2:/opt/scrutiny/influxdb
    devices:
      - "/dev/sda:/dev/sda"
      - "/dev/sdb:/dev/sdb"
      - "/dev/zfs:/dev/zfs"                # needed for ZFS pool monitoring
```

1. Run `smartctl --scan` on the host and add **every** listed device under `devices:`. Scrutiny can only see disks you explicitly pass through. Drive letters (`sda`, `sdb`...) can change between reboots, so pass through every letter in use, not just the ones you care about.
2. `/run/udev` is required so the collector can read device metadata.
3. `SYS_RAWIO` lets `smartctl` query SMART data. `SYS_ADMIN` is only required if you have **NVMe** drives.
4. `COLLECTOR_HOST_ID` is **required** if you use ZFS monitoring. Without it, every pool shows **UNKNOWN** and "Inventory unavailable", even though the pools are healthy. The ZFS collector only sends a complete pool list when it knows which host it is reporting for. It also groups disks per host on the Hosts tab.
5. `TZ` sets the time zone for the cron schedules and for scheduled reports. Without it they run in UTC.
6. Alternatively, `privileged: true` replaces `cap_add` and `devices:` completely, which saves you maintaining the device list.

> Ports: **8080** is the web UI/API, **8086** is InfluxDB. You only need to publish 8086 if something outside the container will query InfluxDB directly.
{.is-info}

> **Upgrading an existing omnibus install:** recent versions moved the bundled InfluxDB to **2.9.1**. Stop Scrutiny and back up the whole `influxdb2` folder, then add `SCRUTINY_INFLUXDB_29_BACKUP_CONFIRMED=true`. The container will not start InfluxDB without it. Fresh installs don't need this.
{.is-warning}

> **The Scrutiny app in the TrueNAS Apps catalog is NOT this version.** The Community train app installs the *original* AnalogJ build, a different project on a different versioning track. It does not include ZFS pool monitoring, Prometheus metrics, Home Assistant MQTT discovery, performance benchmarking, workload insights, or the rebuilt UI. To run the fork on TrueNAS, deploy it as a **Custom App** using the compose file above.
{.is-warning}

## <img src="/docker.png" class="tab-icon"> Docker (Hub/Spoke)

Use this when you have multiple servers and want one dashboard. Run the web + InfluxDB containers once, then the collectors on every machine with disks.

```yaml
services:
  influxdb:
    image: influxdb:2.9.1
    container_name: scrutiny-influxdb
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/scrutiny/influxdb2:/var/lib/influxdb2

  web:
    image: ghcr.io/starosdev/scrutiny:latest-web
    container_name: scrutiny-web
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - SCRUTINY_WEB_INFLUXDB_HOST=influxdb
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/scrutiny/config:/opt/scrutiny/config
    depends_on:
      - influxdb

  collector:
    image: ghcr.io/starosdev/scrutiny:latest-collector
    container_name: scrutiny-collector
    restart: unless-stopped
    cap_add:
      - SYS_RAWIO
      - SYS_ADMIN
    environment:
      - COLLECTOR_API_ENDPOINT=http://web:8080
      - COLLECTOR_HOST_ID=server1            # unique per machine
      - COLLECTOR_CRON_SCHEDULE=0 0 * * *
      - TZ=America/New_York
    volumes:
      - /run/udev:/run/udev:ro
    devices:
      - "/dev/sda:/dev/sda"
      - "/dev/sdb:/dev/sdb"
    depends_on:
      - web

  # Only on machines with ZFS pools
  collector-zfs:
    image: ghcr.io/starosdev/scrutiny:latest-collector-zfs
    container_name: scrutiny-collector-zfs
    restart: unless-stopped
    environment:
      - COLLECTOR_API_ENDPOINT=http://web:8080
      - COLLECTOR_HOST_ID=server1            # same ID as this machine's SMART collector
      - COLLECTOR_ZFS_CRON_SCHEDULE=*/15 * * * *
      - TZ=America/New_York
    devices:
      - "/dev/zfs:/dev/zfs"
    depends_on:
      - web
```

On remote machines, deploy **only** the collector service(s) and point `COLLECTOR_API_ENDPOINT` at the IP of your web container. Give every machine its own `COLLECTOR_HOST_ID`, and use the same ID for that machine's SMART and ZFS collectors.

| Image | Purpose |
|-------|---------|
| `:latest-omnibus` | Everything in one container |
| `:latest-web` | Web UI + API only |
| `:latest-collector` | SMART collector (one per server) |
| `:latest-collector-zfs` | ZFS pool health collector (one per ZFS server) |
| `:latest-collector-performance` | fio benchmark collector (heavy disk I/O, schedule it off-hours) |




# 2 · Configuration

Config files live in `/opt/scrutiny/config` (i.e. `/mnt/tank/configs/scrutiny/config` on the host). None are required, but they unlock everything below.

| File | Purpose |
|------|---------|
| `scrutiny.yaml` | Web app / API settings, notifications, attribute overrides |
| `collector.yaml` | SMART collector settings, device type overrides, labels |
| `collector-zfs.yaml` | ZFS pool collector |
| `collector-performance.yaml` | fio benchmark collector |


Every key can also be set as an environment variable — `SCRUTINY_` prefix for the web app, `COLLECTOR_` for the collector, with dots becoming underscores (`web.listen.port` → `SCRUTINY_WEB_LISTEN_PORT`). Env vars win over the config file.

## 2.1 Collection Schedule

The collector runs daily at midnight by default. Override with the `COLLECTOR_CRON_SCHEDULE` environment variable (standard cron syntax). To force a run right now:

```bash
docker exec scrutiny /opt/scrutiny/bin/scrutiny-collector-metrics run
```

## 2.2 RAID & Device Detection

Scrutiny uses `smartctl --scan` to find drives. If a device type is detected incorrectly (common with RAID controllers), override it in `collector.yaml`. If you use Docker, the virtual disk **must** be visible to the container — it may live under `/dev/*` or `/dev/bus/*`.

> If a drive shows up with no SMART data, the device type is almost always the culprit. Try `COLLECTOR_COMMANDS_METRICS_SMART_ARGS="--xall --json -T permissive"`.
{.is-info}

## 2.3 S.M.A.R.T Attribute Overrides

Noisy attribute triggering false failures? On the device detail page, click the three-dot menu in the **Actions** column and choose **Ignore attribute** or **Force passed**. For global rules, use **Dashboard Settings → SMART Attribute Overrides**, or the `smart.attribute_overrides` block in `scrutiny.yaml`.

| Action | Behavior |
|--------|----------|
| Ignore | Attribute marked passed; excluded from device status and notifications |
| Force Status | Force passed / warn / failed |
| Custom Threshold | Your own `warn_above` / `fail_above` values |


# 3 · Notifications
 
Scrutiny alerts you when a drive's health changes. In this fork, notification endpoints are managed **entirely in the web UI** — add, edit, test, and delete them without touching a config file or restarting the container.
 
## 3.1 Adding an Endpoint
 
1. Open **Settings** in the web UI
2. Under **Notifications**, click **Add**
3. Paste your notification URL
4. Click **Test**, then **Save**
Scrutiny speaks **Shoutrrr**, **Apprise**, custom scripts, and raw webhooks — Discord, Telegram, email, Slack, Teams, ntfy, Gotify, Pushover, and dozens more.
 
- [**Apprise URL Builder**](https://appriseit.com/url-builder/) — URL format builder for every supported service
- [**Shoutrrr docs**](https://nicholas-fedor.github.io/shoutrrr/) — URL format for the built-in Shoutrrr targets
{.links-list}

 
> **Usernames and passwords with special characters must be URL-encoded.** If your username is `myname@example.com` and your password is `124@34$1`, the SMTP URL becomes:
> `smtp://myname%40example%2Ecom:124%4034%241@ms.my.domain.com:587`
{.is-warning}
 
Two gotchas worth knowing:
 
- **Gotify defaults to HTTPS.** Plain-HTTP deployments need `gotify://gotify-host:8080/token?disabletls=Yes`
- **Telegram topic groups** take the thread ID after a colon: `channels=-123456789:12345`

# <img src="/eagle.png" class="tab-icon"> 8 · Would You Like to Know More?


https://blog.serversatho.me/getting-smart-about-your-data-why-drive-health-isnt-enough/



