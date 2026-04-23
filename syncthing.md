---
title: Syncthing
description: A guide to deploying Syncthing
published: true
date: 2026-04-23T10:31:51.612Z
tags: 
editor: markdown
dateCreated: 2026-04-23T10:31:51.612Z
---

# <img src="/syncthing.png" class="tab-icon"> What is Syncthing?

**Syncthing** is a free, open-source, continuous file synchronization program that keeps files in sync between two or more devices in real time. It uses a peer-to-peer mesh model — there is no central server, no cloud account, and no third party brokering your data. All traffic is TLS encrypted device-to-device, devices are mutually authenticated with cryptographic certificates, and files are chunked into blocks so multiple peers can contribute to a sync in parallel.



# 1 · Deploy Syncthing
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

This wiki uses the official upstream `syncthing/syncthing` image maintained by the Syncthing project itself.

```yaml
services:
  syncthing:
    image: syncthing/syncthing:latest
    container_name: syncthing
    hostname: syncthing
    environment:
      - PUID=568
      - PGID=568
      - PCAP=cap_chown,cap_fowner+ep
    volumes:
      - /mnt/tank/configs/syncthing:/var/syncthing
      - /mnt/tank/[additional_data_set]:/data1
    network_mode: host
    restart: unless-stopped
    healthcheck:
      test: curl -fkLsS -m 2 127.0.0.1:8384/rest/noauth/health | grep -o --color=never OK || exit 1
      interval: 1m
      timeout: 10s
      retries: 3
```

| Port | Protocol | Purpose |
|------|----------|---------|
| 8384 | TCP | Web UI |
| 22000 | TCP | Sync protocol (primary) |
| 22000 | UDP | QUIC sync transport |
| 21027 | UDP | Local discovery (LAN broadcast/multicast) |


> 
> **Host networking is strongly recommended** by the Syncthing devs. In bridge mode Syncthing only sees the container's internal `172.x.x.x` address, which breaks LAN peer discovery and can drop transfer rates to WAN speeds even between machines on the same switch. With `network_mode: host` the ports table above is informational — Syncthing binds them directly on the host.
{.is-warning}

> 
> The official image runs Syncthing as **PUID:PGID (568:568 above)**, dropping from root at startup. The `PCAP=cap_chown,cap_fowner+ep` line grants just enough Linux capabilities for Syncthing's **Sync Ownership** folder feature to preserve original file ownership across peers — without running the whole container as root. If you don't care about cross-peer ownership (most homelab cases), you can drop the `PCAP` line and synced files will simply land on disk as `568:568`.
{.is-info}



## <img src="/truenas.png" class="tab-icon"> TrueNAS

The TrueNAS Stable app train includes an official Syncthing app.

1. In the TrueNAS UI go to **Apps → Discover Apps** and search for `Syncthing`
2. Click **Install**
3. Configure the following:
   - **Application Name**: `syncthing`
   - **Timezone**: your local TZ
   - **Storage → Config**: Host Path, `/mnt/tank/configs/syncthing`, enable ACL
   - **Storage → Additional Storage**: add host paths for each dataset you want to sync (e.g. `/mnt/tank/media`)
   - **Network**: accept defaults (8384 / 22000 / 21027)
4. Click **Install** and wait for the app to show **Running**
5. Open the Web UI and set a username/password

> 
> The TrueNAS Stable Syncthing app runs as **root (UID/GID 0)** with elevated capabilities (CHOWN, DAC_OVERRIDE, FOWNER, SETFCAP, SETGID) so it can preserve ownership and ACLs across peers. Make sure the datasets you point at it have an ACL entry for user `0` with full permissions, or Syncthing will fail to write.
{.is-warning}

# 2 · Configuration

## 2.1 Set a Web UI password

Syncthing listens on `0.0.0.0:8384` by default, meaning the Web UI is exposed to anyone on the network until you lock it down. This is the first thing you should do after install.

1. Open the Web UI at `http://<host>:8384`
2. Click **Actions → Settings → GUI**
3. Set **GUI Authentication User** and **GUI Authentication Password**
4. (Optional) Enable **Use HTTPS for GUI**
5. Click **Save** — Syncthing will restart the UI

## 2.2 Rename this device

Under **Actions → Settings → General**, set a meaningful **Device Name** (e.g. `truenas-home`, `desktop-fedora`). This is what other peers will see when you try to pair.

## 2.3 Pair with another device

1. On the remote device, open its Syncthing Web UI and copy its **Device ID** from **Actions → Show ID**
2. On your NAS, click **Add Remote Device**, paste the ID, give it a friendly name, and save
3. Accept the pairing prompt that appears on the remote device
4. You should now see the peer listed as **Connected**

## 2.4 Add a shared folder

1. Click **Add Folder**
2. Set a **Folder Label** (display name) and **Folder ID** (unique identifier — must match on both sides)
3. Set **Folder Path** to the container path you want to sync (e.g. `/data1/photos`)
4. Under the **Sharing** tab, tick the remote device(s) to share with
5. Under **Advanced**, pick a **Folder Type**:
   - **Send & Receive** — true two-way sync
   - **Send Only** — this node is the source of truth
   - **Receive Only** — this node is a read-only mirror
6. Save, then accept the share on the remote device

> 
> Path and file names in Syncthing are **case sensitive**. `MyData.txt` and `mydata.txt` are two different files and can create conflict copies if peers disagree on case (common when syncing between Linux and Windows/macOS).
{.is-info}

## 2.5 File versioning (recommended)

By default a deleted/overwritten file is gone from all peers. Enable versioning per folder under **Folder → Advanced → File Versioning**:

| Mode | Behavior |
|------|----------|
| Trash Can | Deleted files go to `.stversions/` for N days |
| Simple | Keep N old versions per file |
| Staggered | More versions for recent changes, fewer for old ones |
| External | Hand off to an external command/script |


Staggered is a good default for "actual backup" intent; Trash Can is fine for everyday working folders.

## 2.6 Ignore patterns

Every folder has a `.stignore` file (edit via **Folder → Edit → Ignore Patterns**). Typical entries for a homelab:

```
.DS_Store
Thumbs.db
*.tmp
*.part
.stversions
node_modules
```

# 3 · Troubleshooting

- **Peer shows "Disconnected" but both sides are online** — check that port `22000/tcp` and `22000/udp` are reachable on the host. Use **Actions → Show ID → Discovery** to see what addresses Syncthing is advertising.
- **"Permission denied" writing to a folder on TrueNAS** — the dataset ACL is missing an entry for UID `0` (Stable app) or `568` (Docker). Re-apply ACLs.
- **Ridiculous database migration on first 2.x launch** — expected, let it run. Watch progress in the container logs.
- **GUI unreachable from other machines** — make sure `STGUIADDRESS` is not set to an empty string in your compose. Leave it unset so the image's default of `0.0.0.0:8384` applies.
- **Synced file ownership is all `568:568` instead of the original UID** — `PCAP=cap_chown,cap_fowner+ep` is missing from the environment block, OR the folder's **Sync Ownership** toggle is not enabled under **Folder → Advanced**. Both are required.

