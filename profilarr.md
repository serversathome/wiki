---
title: Profilarr
description: A guide to deploying Profilarr with docker compose
published: true
date: 2026-10-21T14:40:21.541Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:30.974Z
---

> Having a tough time? [Book time with me](https://links.serversatho.me/hire) 1-on-1 for some help!
{.is-success}

# <img src="/profilarr.png" class="tab-icon"> What is Profilarr?

**Profilarr** syncs quality profiles, custom formats, and media management settings from shared configuration databases into your Radarr and Sonarr instances. Instead of hand-copying custom formats from forum posts, pasting JSON, and re-doing it all the next time TRaSH Guides updates, you link one (or more) curated databases in Profilarr, customize what you need, then push everything to any number of Arr instances.

v2 is a complete rewrite with a new database and customization model, library pages for both Radarr and Sonarr, simultaneous multi-database support, drift detection, built-in upgrade automation (a Huntarr / Upgradinatorr replacement), Renameinatorr-style rename automation, Health Checkarr-style cleanup, in-app onboarding, in-app announcements, OIDC support, scoped API keys, and an overhauled testing engine.

> 
> v2 is **not** compatible with v1. The underlying database and customization systems changed significantly, so existing v1 databases, configs, and appdata won't work in v2. If you're upgrading from v1, treat this as a fresh install on a new config volume.
{.is-warning}

> 
> Profilarr v2 requires a Docker host running Linux kernel 3.17 or newer on `amd64` or `arm64`. Older kernels (including Synology DSM installs on kernel 3.10) are **not** supported. It also requires **Sonarr v4+** and **Radarr v5+**. Sonarr v5 is supported on an **experimental** basis as of v2.3.0.
{.is-danger}

<div class="glance">
  <div><span>Port</span><b><code>6868</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service + optional parser</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/dictionarry-hub/profilarr"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://youtu.be/jvir2SmnwWs"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

# 1 · Deploy Profilarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  profilarr:
    image: ghcr.io/dictionarry-hub/profilarr:latest
    container_name: profilarr
    restart: unless-stopped
    ports:
      - "6868:6868"
    volumes:
      - /mnt/tank/configs/profilarr:/config
    environment:
      - PUID=568
      - PGID=568
      - UMASK=022
      - TZ=America/New_York
      # Uncomment ORIGIN if accessing through a reverse proxy (required for OIDC)
      #- ORIGIN=https://profilarr.yourdomain.com
      - PARSER_HOST=parser
      - PARSER_PORT=5000
    depends_on:
      parser:
        condition: service_healthy

  # Optional: only needed for custom format and quality profile testing
  parser:
    image: ghcr.io/dictionarry-hub/profilarr-parser:latest
    container_name: profilarr-parser
    restart: unless-stopped
    user: "568:568"
    expose:
      - "5000"
```

> 
> The **parser** service only powers the custom format and quality profile **testing** tools. Linking, syncing, upgrades, and everything else work without it. If you don't need testing, delete the `parser` service along with the `PARSER_HOST`, `PARSER_PORT`, and `depends_on` lines. If you keep it, run both images on the same tag.
{.is-info}


## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
2. Search for **Profilarr** (Community train; app version 2.0.2 or newer ships Profilarr 2.0.4+)
3. Click **Install**
4. Configure the following settings:
   - **Application Name**: `profilarr`
   - **Host Path** for `/config`: `/mnt/tank/configs/profilarr`
   - **WebUI Port**: `6868`
   - **Timezone**: `America/New_York`
5. Click **Install**



> 
> **Upgrading to v2.3.0 or newer from an earlier v2 release?** Two behavior changes:
> - The **local network auth bypass is gone**. If you used it to skip the login on your LAN, you'll now get a login prompt. Sign in with your Profilarr username and password.
> - Any **existing API keys become full-access "default" keys**. If you've handed a key to another tool, consider replacing it with a scoped key (see section 2).
>
> v2.3.0 also fixes three security advisories, so update if you're on an older version.
{.is-warning}



# 2 · First Login

1. Open `http://your-server:6868` in your browser
2. On first launch, Profilarr's in-app onboarding walks you through everything. Set your **username and password** when prompted and the wizard handles the rest
3. (Optional) Authentication is controlled by the `AUTH` environment variable:
   - `AUTH=on` (the default) uses username and password login. To add **SSO** with your IdP (Authelia, Authentik, Google, etc.), keep `AUTH=on` and also set `OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET`, `OIDC_DISCOVERY_URL`, and `ORIGIN`. Password and SSO login then work side by side. The redirect URL is `{ORIGIN}/auth/oidc/callback`
   - `AUTH=off` to disable auth entirely, only if Profilarr sits behind your own reverse proxy that handles authentication

> 
> `AUTH=oidc` is **deprecated** as of v2.3.0. If your compose still uses it, switch to `AUTH=on` with all three `OIDC_*` settings.
{.is-warning}

You can re-run the onboarding flow any time from **Settings → Onboarding**, and pick which page Profilarr opens to by default.

## 2.1 API Keys

Other tools (dashboards, MCP servers, scripts) talk to Profilarr through an API key. You can create multiple **named keys**, each with **per-area permissions**, so a read-only dashboard doesn't get the same access as a script that triggers syncs. Profilarr asks you to confirm your identity before it creates a key.

Give each tool its own key with only the permissions it needs, so you can revoke one without breaking the others.



# 3 · Link a Database

v2 supports **multiple databases at once**, so you can mix profiles from different sources. Navigate to **Databases** on the left sidebar and click **+** to **Link Repository**.

| Database | Repo | Notes |
|----------|------|-------|
| Dictionarry | `https://github.com/Dictionarry-Hub/database` | The default. Connected automatically. Covers 720p through 2160p, from compact x265 encodes to UHD remuxes |
| TRaSH PCD | `https://github.com/Dictionarry-Hub/trash-pcd` | A port of the TRaSH Guides in PCD format. Maintained by the Dictionarry team (not the TRaSH team), so report any issues to Dictionarry first. French and German profiles still in progress |
| Dumpstarr | `https://github.com/Dumpstarr/Database` | Community fork built on top of Dictionarry and TRaSH formats |



# 4 · Link Your Arrs

1. Navigate to **Arrs** on the left sidebar
2. Click **+** to add a new instance
3. Fill in:
   - **Name**: e.g. `Radarr`, `Radarr 4K`, `Sonarr`
   - **Type**: Radarr or Sonarr
   - **URL**: `http://radarr:7878` (or whatever IP/host:port works on your network)
   - **API Key**: from Radarr/Sonarr → **Settings → General → API Key**
   - **Basic Auth** (optional): username and password, only if the Arr sits behind a reverse proxy that requires a login
4. Click **Test** to verify, then **Save** at the top right

Repeat for every Arr instance you want Profilarr to manage. v2 cleanly handles cloned 1080p / 4K Radarr setups even when they share an API key.

> 
> If the connection test fails and the Arr is behind a reverse proxy with its own login page, Profilarr now tells you so. Either point the URL at the Arr directly (e.g. its container name and port) or fill in the Basic Auth fields. Also note that **changing an instance's URL later requires re-entering its API key**.
{.is-info}

## 4.1 Library Pages

Each linked Arr in v2 gets a full **library page** inside Profilarr: table and card views over your movies or series with configurable display fields, so you can browse and operate on the library without bouncing back to Radarr/Sonarr.

- **Smart filters** with AND logic, negation, and range queries across quality, profile, year, genre, status, monitored, etc.
- **Filter by custom format** to show only items where a given custom format does (or doesn't) apply
- **Sort by custom format score** to surface your worst-scoring items at the top so you know what to upgrade
- Configurable columns / card fields per view. Your sort order and download status filter are remembered between visits

This is the easiest way to spot mis-tagged or mis-scored items before they bite you on the next upgrade run.

## 4.2 Sync to Your Arrs

Sync is triggered per Arr from the **Arrs** view (or scheduled, see section 7).

1. Open an Arr in the **Arrs** view
2. Review what's queued to push (Profilarr shows a diff)
3. Click **Sync** to apply

After syncing, head into Radarr/Sonarr and:

1. Open **Movies** / **Series**
2. Select all → **Edit → Quality Profile** → pick your new Profilarr profile
3. Trigger an **Interactive Search** or **Cut-off Unmet** search to upgrade your existing files, or let Profilarr's Upgrades (section 4.4) work through them over time

## 4.3 Arr Drift Detection

Profilarr periodically checks each linked Arr against the configuration Profilarr last synced to it. If something has drifted (a custom format edited directly in Radarr, a score changed in Sonarr's UI, a quality profile renamed by hand, a delay profile or media management setting nudged), Profilarr surfaces it as a **drift event**.

This catches the silent failure mode where your 4K Radarr instance slowly diverges from your 1080p one because someone tweaked a score in the Radarr UI and forgot.

Drift events fire notifications and can be reconciled by re-syncing from Profilarr.

## 4.4 Upgrades

The Arrs react well to new releases via RSS, but they don't continuously revisit older downloads looking for something better. That matters most when you switch or update quality profiles and end up with files the new profile would no longer have grabbed. **Upgradinatorr** solved that by cycling through the library and triggering searches over time. Profilarr v2 brings the same idea inside, with a GUI and more control:

- **Filter** by any metadata your Arr tracks: ratings, year, genre, size, release group, language, date added, and more
- Filters support **nested AND/OR** logic
- **Selectors** let you prioritize what gets searched first
- **Cooldowns** prevent the same item from being repeatedly hammered
- Runs on a schedule

Tune **Count** (how many items to process per run) and the match filters if you want to be more specific. The defaults work well.

### Sample Filters

These are the filters I run. Each one targets monitored items that haven't hit their profile's cutoff yet and searches the lowest scoring ones first.

<details>
<summary><b>Radarr: Servers@Home Upgrade Filter</b></summary>

Matches movies that are monitored, released, and below cutoff. Searches the 5 lowest scoring movies per run.

```json
{
  "name": "Servers@Home Upgrade Filter",
  "enabled": true,
  "group": {
    "type": "group",
    "match": "all",
    "children": [
      {
        "type": "rule",
        "field": "monitored",
        "operator": "is",
        "value": true
      },
      {
        "type": "rule",
        "field": "status",
        "operator": "eq",
        "value": "released"
      },
      {
        "type": "rule",
        "field": "cutoff_met",
        "operator": "is",
        "value": false
      }
    ]
  },
  "selector": "lowest_score",
  "count": 5,
  "cutoff": 100,
  "tag": ""
}
```

</details>

<details>
<summary><b>Sonarr: Servers@Home Upgrade Filter</b></summary>

Matches series that are monitored, either continuing or ended, below cutoff, and that have at least one episode file on disk. Searches 1 series per run.

```json
{
  "name": "Servers@Home Upgrade Filter",
  "enabled": true,
  "group": {
    "type": "group",
    "match": "all",
    "children": [
      {
        "type": "rule",
        "field": "monitored",
        "operator": "is",
        "value": true
      },
      {
        "type": "group",
        "match": "any",
        "children": [
          {
            "type": "rule",
            "field": "status",
            "operator": "eq",
            "value": "continuing"
          },
          {
            "type": "rule",
            "field": "status",
            "operator": "eq",
            "value": "ended"
          }
        ]
      },
      {
        "type": "rule",
        "field": "cutoff_met",
        "operator": "is",
        "value": false
      },
      {
        "type": "rule",
        "field": "episode_file_count",
        "operator": "gt",
        "value": 0
      }
    ]
  },
  "selector": "lowest_score",
  "count": 1,
  "cutoff": 100,
  "tag": ""
}
```

</details>

> 
> Don't run two upgrade services at the same time. If you're already using Huntarr, Upgradinatorr, or another upgrade tool and you're happy with it, leave Profilarr's Upgrades disabled. Running both will fight each other and burn through indexer queries. This feature is optional if you already use a replacement service you like.
{.is-warning}

## 4.5 Rename & Cleanup

- **Rename**: bulk rename files and folders with **dry-run previews** before anything touches disk. Inspired by Renameinatorr. v2 also adds an `include-in-rename` flag at the database spec level so each renamable item is tracked properly.
- **Arr Cleanup**: periodic housekeeping that removes orphaned entries and stale config bits from each Arr. Inspired by Health Checkarr.

Both are configured per Arr. Cleanup can be scheduled via Jobs. As of v2.3.0, rename **runs** are separate from the rename **schedule**, so you can kick off a one-off rename (or dry run) without changing or waiting on the scheduled job.

# 5 · Build & Customize

## 5.1 Quality Profiles

- Group and order qualities (e.g. WEB-DL 1080p → Bluray 1080p → Remux)
- Assign custom format scores **per app**, so Radarr and Sonarr can score the same format differently
- Profiles inherit defaults from the linked database, with your local customizations layered on top and preserved across upstream updates
- With **Sonarr v5** (experimental), quality sizes can be set **per profile** instead of only globally

## 5.2 Custom Formats

- Match releases by resolution, source, release group, file size, language, codec, HDR flag, audio channels, and more
- Conditions are composable, so you can build complex rules without leaving the UI
- v2 also adds per-condition Arr types so the same format can behave differently on Radarr vs Sonarr where needed
- A condition that uses a regex shows the pattern and links straight to it

## 5.3 Regular Expressions

- Reusable regex patterns shared across multiple custom formats
- Edit a pattern once and every custom format that references it updates
- Patterns can carry an attached **Regex101 link**. Profilarr parses the test cases from it so you have inline validation



# 6 · Media Management & Delay Profiles

Under **Media Management** in the sidebar you'll find:

- **Naming Settings**: file and folder naming schemes
- **Quality Definitions**: size limits per quality (your Bluray-2160p ceiling, etc.)
- **Media Settings**: Radarr/Sonarr's misc media options

And separately, **Delay Profiles** is its own config type in v2, controlling protocol preferences (usenet vs torrent), release delays, and score gates. Delays display in minutes, hours, or days, so a 1440 minute delay reads as 1 day.

> 
> A v2 change worth noting: Media Management configs are no longer one-per-instance. You can have multiple quality definitions, naming schemes, and media settings, and pick which one each Arr uses.
{.is-info}

Set these to your database of choice (typically Dictionarry's defaults) and let Profilarr keep them aligned across instances. The defaults are good and don't need to be adjusted unless you have a specific use case.



# 7 · Jobs (Scheduler)

**Settings → Jobs** is where everything schedulable lives:

- **Database refresh**: pull upstream database updates (Dictionarry pushes new formats over time as codecs and release groups evolve)
- **Arr Sync**: push config to your Arrs on a cadence
- **Arr Cleanup**: periodic library cleanup
- **Upgrades**: the scheduled upgrade search
- **Renames**: periodic library renames (one-off runs can still be started separately, see section 4.5)
- **Drift detection**: how often to check Arrs for drift
- **Backups**: config backup snapshots (also accessible under **Settings → Backups**)

Repo sync and backup intervals are tunable from the Jobs page (added in v2.0.x).

A reasonable starting cadence: database refresh every few hours, Arr Sync daily, drift detection daily, backups daily.

# 8 · Notifications

**Settings → Notifications**. Profilarr v2 supports these notification services:

- **Discord** (webhook with optional bot username, avatar, and @here mentions)
- **Telegram**
- **Slack**
- **Ntfy**
- **Pushover**
- **Gotify**
- **Apprise**
- **Webhook** (generic)

When you create a service you can scope which event categories it receives. Events are grouped into:

| Category | Events |
|----------|--------|
| Backups | Backup Completed, Backup Failed |
| Databases | Database Linked, Database Unlinked, Database Updates Available, Database Synced, Database Sync Failed, Database Link Failed |
| Arr Sync | Arr Sync Completed (Success / Partial), Arr Sync Failed |
| Arr Cleanup | Arr Cleanup Completed (Success / Partial), Arr Cleanup Failed |
| Arr Drift | Arr Drift Detected, Arr Drift Failed |
| Upgrades | Upgrade Completed (Success / Partial), Upgrade Failed |
| Renames | Rename Completed (Success / Partial), Rename Failed |
| Announcements | New Announcement |


The **All / Success / Failed** toggles at the top of the picker let you bulk-enable common selections.



# <img src="/youtube.png" class="tab-icon"> 9 · Video
https://youtu.be/jvir2SmnwWs