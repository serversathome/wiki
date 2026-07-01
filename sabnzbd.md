---
title: SABnzbd
description: A guide to deploying SABnzbd via TrueNAS or docker
published: true
date: 2026-07-01T18:54:24.548Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:11.759Z
---

# <img src="/sabnzbd.png" class="tab-icon"> What is SABnzbd?

**SABnzbd** is a free, open-source Usenet download client, sometimes called a binary newsreader. You hand it an **NZB file** and it does everything else automatically: downloads all the article pieces from your Usenet provider, verifies them, repairs anything missing with PAR2, extracts the archives, and drops the finished file into your library.

Think of it as the Usenet counterpart to a torrent client like qBittorrent. It's also the standard download engine that sits underneath a Sonarr and Radarr setup, so most people run it fully automated and rarely open it by hand.

> To use SABnzbd you need two other things it does **not** provide: a **Usenet provider** (where the files actually live, e.g. Newshosting) and an **indexer** (a search engine that gives you NZB files, e.g. SceneNZBs). Only the provider is entered into SABnzbd. The indexer feeds NZBs in from the outside, usually through Sonarr/Radarr.
{.is-info}

# 1 · Deploy SABnzbd
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:5.0.4
    container_name: sabnzbd
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/sabnzbd:/config
      - /mnt/tank/media:/data
    ports:
      - "8080:8080"
    restart: unless-stopped
```

1. If your pool is named something other than `tank`, change the left side of both volume paths.
2. The `/data` mount should be the **parent of both your downloads and your media library** (e.g. `/mnt/tank/media` holds `downloads/`, `movies/`, `tv/`). Mapping one shared folder is what lets Sonarr and Radarr hardlink finished downloads instead of copying them.
3. `PUID`/`PGID` `568` is the `apps` user on TrueNAS, matching the recommended dataset permissions.

> Map the **same host folder to the same container path** in SABnzbd, Sonarr, and Radarr (`/data` everywhere). Mismatched paths are the number-one cause of imports silently failing.
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI.
2. Click **Discover Apps** and search for **SABnzbd** (Community train).
3. Click **Install**.
4. Configure the following:
   - **App Name**: `sabnzbd`
   - **WebUI Port**: `8080`
   - **Config Storage**: Host Path → `/mnt/tank/configs/sabnzbd`
   - **Additional Storage**: Host Path `/mnt/tank/media` → Mount Path `/data` (your downloads + media dataset)
5. Click **Install** and wait for the app to report **Running**.

> The TrueNAS SABnzbd app runs as UID/GID **568** (`apps`) by default, so files it creates are owned correctly for the rest of your media stack.
{.is-info}

# 2 · Configuration

## 2.1 Add your Usenet provider

On first launch, the setup wizard asks for your provider. You can also add it later under **Config → Servers**.

| Field | Value |
|-------|-------|
| Host | `news.newshosting.com` |
| Port | `563` |
| SSL | ✅ Enabled |
| Username / Password | Your provider account credentials |
| Connections | `30`–`50` to start (Newshosting allows up to 100) |


Click **Test Server**. When it turns green, SABnzbd can reach your provider. This is the only server you need to add.

> Always use SSL on port `563`. It keeps your provider connection encrypted. More connections can raise speed up to your plan's limit, but there are diminishing returns and it uses more CPU, so don't max it out on day one.
{.is-warning}

## 2.2 Set your download folders

Go to **Config → Folders**. Two fields matter:

- **Temporary Download Folder** — SABnzbd's scratch space for in-progress articles, repair, and unpack.
- **Completed Download Folder** — where finished, repaired, unpacked files land.

They can't be the exact same path, but they can be two subfolders under one downloads folder on your media dataset:

| Folder | Value |
|--------|-------|
| Temporary Download Folder | `/data/downloads/incomplete` |
| Completed Download Folder | `/data/downloads/complete` |
{.dense}

> Keeping the completed folder on the **same dataset as your media library** lets Sonarr/Radarr **hardlink** on import (instant, no extra disk). Keeping the incomplete folder there too makes SABnzbd's own incomplete → complete move an instant rename instead of a copy.
{.is-success}

## 2.3 Categories (for Sonarr / Radarr)

Categories are optional, Sonarr and Radarr import by reading the finished path from SABnzbd's API, so a bare setup works. But when both apps share one SABnzbd, categories keep each app scoped to its own downloads and its own folder.

1. Go to **Config → Categories**.
2. Add a category `tv` and, in its folder field, type the plain relative name `tv`.
3. Add a category `movies` with folder `movies`.

SABnzbd nests each one under your Completed Download Folder automatically, so `tv` becomes `/data/downloads/complete/tv`. You don't pre-create the folders or re-point SABnzbd at them.

> Use a **plain relative name** (`tv`), not a full path. An absolute path in the folder field overrides the completed folder and sends that category elsewhere.
{.is-warning}

> **Categories vs Tags:** SABnzbd calls these *categories*. Sonarr/Radarr have a separate, unrelated feature called *tags* (release profiles, indexer restrictions, etc.) that you do **not** need for a normal setup. The only field that must match between the *arrs and SABnzbd is the download-client **Category**.
{.is-info}

## 2.4 Connect Sonarr & Radarr

Grab your API key from **Config → General → API Key**, then in each *arr:

1. Go to **Settings → Download Clients → + → SABnzbd**.
2. **Host**: SABnzbd's IP or hostname &nbsp;·&nbsp; **Port**: `8080`
3. **API Key**: paste the key from above.
4. **Category**: `tv` in Sonarr, `movies` in Radarr.
5. **Test**, then **Save**.

> Make sure SABnzbd, Sonarr, and Radarr all resolve `/data` to the same host folder. If SAB reports a completed path the *arrs can't see, imports fail even though the download succeeded.
{.is-danger}

## 2.5 Connect to Prowlarr
 
Prowlarr is the hub of the automation, it manages your indexers (like SceneNZBs) and pushes them out to Sonarr and Radarr. Adding SABnzbd as a download client in Prowlarr lets you send releases straight to it from Prowlarr's own interactive search.
 
First grab SABnzbd's API key from **Config → General → API Key**. Then in Prowlarr:
 
1. Go to **Settings → Download Clients → + → SABnzbd**.
2. **Host**: SABnzbd's IP or hostname &nbsp;·&nbsp; **Port**: `8080`
3. **API Key**: paste the key from above.
4. **Category**: optional here, leave blank or set a catch-all.
5. **Test**, then **Save**.



# 3 · Advanced Tweaks (Optional)

Everything below is optional, but it makes SABnzbd tidier and safer for an automated media setup.

> Turn on **Advanced Settings** (the toggle at the top of each Config page) to reveal the advanced fields, several of the settings below live there.
{.is-warning}

## 3.1 Recommended Switches

Found under **Config → Switches**:

| Setting | Recommended | Why |
|---------|-------------|-----|
| **Direct Unpack** | On | Unpacks during the download instead of after, cutting time on large releases. Disable only if your CPU or disk is weak. |
| **Ignore Samples** | On | Automatically removes sample video clips from finished downloads. |
| **Abort jobs that cannot be completed** | On | Stops grinding on a release once too many articles are missing to ever repair it. |
| **Permissions** | `770` | Sets completed-file permissions to match your media share's group mask. |


## 3.2 Post-Processing Cleanup
 
Under **Config → Switches → Cleanup List**, add file extensions that SABnzbd should delete once a download finishes. This strips junk and risky executables out of your completed releases:
 
```
exe, bat, cmd, com, scr, pif, hta, vbs, js, jar, wsf, ps1, msi, msp, cpl, ad, apk, dll, bin, gadget, vb, vbe, ws, wsc, wsh, lnk, iso, img, dmg, zipx, psm1, psd1, psc1, sh, rb, perl, py, pyd, url
```
 
> **Stronger option:** for the executable extensions, use **Unwanted Extensions** instead of (or in addition to) the Cleanup List, and set **Action when unwanted extension detected** to **Fail job (move to History)**. That rejects the whole release, a classic sign of a fake or malicious NZB, rather than downloading it and deleting the file afterward.
{.is-info}
 
> Don't apply this list globally if you also use SABnzbd to download games or software, it will delete legitimate installers (`exe`, `msi`, `iso`, `dmg`, etc.). Give those their own download-only category instead.
{.is-warning}


# <img src="/patreon-light.png" class="tab-icon"> 4 · Video
