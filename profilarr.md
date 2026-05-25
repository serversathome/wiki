---
title: Profilarr
description: A guide to deploying Profilarr with docker compose
published: true
date: 2026-05-25T07:15:46.963Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:30.974Z
---

# <img src="/profilarr.png" class="tab-icon"> What is Profilarr?

**Profilarr** syncs quality profiles, custom formats, and media management settings from shared configuration databases into your Radarr and Sonarr instances. Instead of hand-copying custom formats from forum posts, pasting JSON, and re-doing it all the next time TRaSH Guides updates, you link one (or more) curated databases in Profilarr, customize what you need, then push everything to any number of Arr instances.

v2 is a complete rewrite with a new database / customisation model, library pages for both Radarr and Sonarr, simultaneous multi-database support, drift detection, built-in upgrade automation (a Huntarr / Upgradinatorr replacement), Renameinatorr-style rename automation, Health Checkarr-style cleanup, in-app onboarding, in-app announcements, OIDC support, and an overhauled testing engine.

> 
> v2 is **not** compatible with v1. The underlying database and customisation systems changed significantly, so existing v1 databases, configs, and appdata won't work in v2. If you're upgrading from v1, treat this as a fresh install on a new config volume.
{.is-warning}

> 
> Profilarr v2 requires a Docker host running Linux kernel 3.17 or newer. Older kernels (including Synology DSM installs on kernel 3.10) are **not** supported.
{.is-danger}

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
      # Uncomment ORIGIN if accessing through a reverse proxy
      #- ORIGIN=https://profilarr.yourdomain.com

```


## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
2. Search for **Profilarr** (Community train, version 2.0.2 or newer ships app version 2.0.4+)
3. Click **Install**
4. Configure the following settings:
   - **Application Name**: `profilarr`
   - **Host Path** for `/config`: `/mnt/tank/configs/profilarr`
   - **WebUI Port**: `6868`
   - **Timezone**: `America/New_York`
5. Click **Install**



# 2 · First Login

1. Open `http://your-server:6868` in your browser
2. On first launch, Profilarr's in-app onboarding walks you through everything — set your **username and password** when prompted and the wizard handles the rest
3. (Optional) Under **Settings → Security**, configure **OIDC** if you want SSO with your IdP (Authelia, Authentik, Google, etc.), or disable auth entirely if you're running Profilarr behind your own reverse proxy that handles authentication

You can re-run the onboarding flow any time from **Settings → Onboarding**.


# 3 · Link a Database

v2 supports **multiple databases at once**, so you can mix profiles from different sources. Navigate to **Databases** on the left sidebar and click **+** to **Link Repository**.

| Database | Repo | Notes |
|----------|------|-------|
| Dictionarry | `github.com/Dictionarry-Hub/database` | The default. Connected automatically. Covers 720p through 2160p, from compact x265 encodes to UHD remuxes |
| TRaSH PCD | `github.com/Dictionarry-Hub/trash-pcd` | A port of the TRaSH Guides in PCD format. Maintained by the Dictionarry team (not the TRaSH team) — report any issues to Dictionarry first. French and German profiles still in progress |
| Dumpstarr | `github.com/Dumpstarr/Database` | Community fork built on top of Dictionarry and TRaSH formats |
| Servers@Home | `github.com/serversathome/profilarr` | Combination of Dictionarry and Dumpstarr |


> 
> As of Profilarr v2.0.2, default links to the official Dictionarry database use the `v2` branch automatically when no branch is specified.
{.is-info}

> 
> The old `serversathome/profilarr` fork (which existed in v1 to add an anime profile) is **deprecated** and will be removed before July 1, 2026. In v2 you don't need it — link the official Dictionarry database alongside any community anime database you want. The TRaSH PCD anime profile is the most established option until Dictionarry ships its own (per-series anime profiling, SeaDex-style, is on the v2.x roadmap).
{.is-warning}

# 4 · Link Your Arrs

1. Navigate to **Arrs** on the left sidebar
2. Click **+** to add a new instance
3. Fill in:
   - **Name**: e.g. `Radarr`, `Radarr 4K`, `Sonarr`
   - **Type**: Radarr or Sonarr
   - **URL**: `http://radarr:7878` (or whatever IP/host:port works on your network)
   - **API Key**: from Radarr/Sonarr → **Settings → General → API Key**
4. Click **Test** to verify, then **Save** at the top right

Repeat for every Arr instance you want Profilarr to manage. v2 cleanly handles cloned 1080p / 4K Radarr setups even when they share an API key.

# 5 · Build & Customize

The build side of Profilarr is where you choose what to actually send to your Arrs. Everything here lives in the Profilarr UI and only touches your Arrs when you sync.

## 5.1 Quality Profiles

- Group and order qualities (e.g. WEB-DL 1080p → Bluray 1080p → Remux)
- Assign custom format scores **per app** — Radarr and Sonarr can score the same format differently
- Profiles inherit defaults from the linked database, with your local customisations layered on top

## 5.2 Custom Formats

- Match releases by resolution, source, release group, file size, language, codec, HDR flag, audio channels, and more
- Conditions are composable — build complex rules without leaving the UI
- v2 also adds per-condition Arr types so the same format can behave differently on Radarr vs Sonarr where needed

## 5.3 Regular Expressions

- Reusable regex patterns shared across multiple custom formats
- Edit a pattern once and every custom format that references it updates
- Patterns can carry an attached **Regex101 link** — Profilarr parses the test cases from it so you have inline validation

## 5.4 Customisations (the v2 change layer)

v1 used a git three-way merge to preserve your tweaks across upstream updates. It worked, but conflicts were painful. v2 replaces it with a dedicated **change layer**: your local changes live separately from the upstream database, so updates can land without overwriting your tweaks or forcing you through merge conflicts. Fewer conflicts surface in the first place, and the ones that do can often resolve automatically.

- Change a score, rename a profile, tweak a regex — the original database file is left alone
- View your customisations in one place under each entity
- Reset to upstream defaults at any time

# 6 · Media Management & Delay Profiles

Under **Media Management** in the sidebar you'll find:

- **Naming Settings** — file and folder naming schemes
- **Quality Definitions** — size limits per quality (your Bluray-2160p ceiling, etc.)
- **Media Settings** — Radarr/Sonarr's misc media options

And separately, **Delay Profiles** is its own config type in v2, controlling protocol preferences (usenet vs torrent), release delays, and score gates.

> 
> A v2 change worth noting: Media Management configs are no longer one-per-instance. You can have multiple quality definitions, naming schemes, and media settings, and pick which one each Arr uses.
{.is-info}

Set these to your database of choice (typically Dictionarry's defaults) and let Profilarr keep them aligned across instances. The defaults are good and don't need to be adjusted unless you have a specific use case.

# 7 · Library Pages

Each linked Arr in v2 gets a full **library page** inside Profilarr — table and card views over your movies or series with configurable display fields, so you can browse and operate on the library without bouncing back to Radarr/Sonarr.

- **Smart filters** with AND logic, negation, and range queries across quality, profile, year, genre, status, monitored, etc.
- **Filter by custom format** — show only items where a given custom format does (or doesn't) apply
- **Sort by custom format score** — surface your worst-scoring items at the top so you know what to upgrade
- Configurable columns / card fields per view

This is the easiest way to spot mis-tagged or mis-scored items before they bite you on the next upgrade run.

# 8 · Sync to Your Arrs

Sync is triggered per Arr from the **Arrs** view (or scheduled — see section 12).

1. Open an Arr in the **Arrs** view
2. Review what's queued to push (Profilarr shows a diff)
3. Click **Sync** to apply

After syncing, head into Radarr/Sonarr and:

1. Open **Movies** / **Series**
2. Select all → **Edit → Quality Profile** → pick your new Profilarr profile
3. Trigger an **Interactive Search** or **Cut-off Unmet** search to upgrade your existing files

# 9 · Arr Drift Detection

A v2 feature: Profilarr periodically checks each linked Arr against the configuration Profilarr last synced to it. If something has drifted — a custom format edited directly in Radarr, a score changed in Sonarr's UI, a quality profile renamed by hand, a delay profile or media management setting nudged — Profilarr surfaces it as a **drift event**.

This catches the silent failure mode where your 4K Radarr instance slowly diverges from your 1080p one because someone (you, three weeks ago) tweaked a score in the Radarr UI and forgot.

Drift events fire notifications (see section 13) and can be reconciled by re-syncing from Profilarr.

# 10 · Upgrades

The Arrs react well to new releases via RSS, but they don't continuously revisit older downloads looking for something better — especially relevant when you switch or update quality profiles and end up with files the new profile would no longer have grabbed. **Upgradinatorr** solved that by cycling through the library and triggering searches over time. Profilarr v2 brings the same idea inside, with a GUI and more control:

- **Filter** by any metadata your Arr tracks: ratings, year, genre, size, release group, language, date added, and more
- Filters support **nested AND/OR** logic
- **Selectors** let you prioritise what gets searched first
- **Cooldowns** prevent the same item from being repeatedly hammered
- The whole thing runs on a schedule

Tune **Count** (how many items to process per run) and the match filters if you want to be more specific — defaults work well.

> 
> Don't run two upgrade services at the same time. If you're already using Huntarr, Upgradinatorr, or another upgrade tool and you're happy with it, leave Profilarr's Upgrades disabled — running both will fight each other and burn through indexer queries. This feature is optional if you already use a replacement service you like.
{.is-warning}

# 11 · Rename & Cleanup

- **Rename** — bulk rename files and folders with **dry-run previews** before anything touches disk. Inspired by Renameinatorr. v2 also adds an `include-in-rename` flag at the database spec level so each renamable item is tracked properly.
- **Arr Cleanup** — periodic housekeeping that removes orphaned entries and stale config bits from each Arr. Inspired by Health Checkarr.

Both are configured per Arr and can be scheduled via Jobs.

# 12 · Jobs (Scheduler)

**Settings → Jobs** is where everything schedulable lives:

- **Database refresh** — pull upstream database updates (Dictionarry pushes new formats over time as codecs and release groups evolve)
- **Arr Sync** — push config to your Arrs on a cadence
- **Arr Cleanup** — periodic library cleanup
- **Upgrades** — the scheduled upgrade search
- **Renames** — periodic library renames
- **Drift detection** — how often to check Arrs for drift
- **Backups** — config backup snapshots (also accessible under **Settings → Backups**)

Repo sync and backup intervals are tunable from the Jobs page (added in v2.0.x).

A reasonable starting cadence: database refresh every few hours, Arr Sync daily, drift detection daily, backups daily.

# 13 · Notifications

**Settings → Notifications**. Profilarr v2 ships with four notification service types:

- **Discord** (webhook with optional bot username, avatar, and @here mentions)
- **Ntfy**
- **Webhook** (generic)
- **Telegram**

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
{.dense}

The **All / Success / Failed** toggles at the top of the picker let you bulk-enable common selections.

# 14 · Announcements

The **Announcements** item in the sidebar surfaces in-app messages from the Profilarr team and database maintainers — release notes, breaking changes, database alerts, etc. — so you don't need to live on Discord or Reddit to keep up. You can subscribe a notification service to the **Announcements** category if you want them pushed.

# 15 · Testing (Optional Parser)

If you deployed the parser sidecar, three powerful testing features unlock:

## 15.1 Regex Testing

Validate your regex patterns against test releases inline in the UI. Patterns can carry an attached Regex101 link — Profilarr parses the test cases from it automatically.

## 15.2 Custom Format Testing

Paste a release name and Profilarr breaks down which conditions in a custom format passed and which failed, with match visualization. Because the parser is the same C# engine the Arrs use, what you see here is what Radarr/Sonarr will see at grab time.

## 15.3 Quality Profile Simulator

Store interactive search results and test them against **all** of your quality profiles at once. See exactly which release each profile would pick, and which scores broke the tie — *before* deploying.

# 16 · Useful Links

- [Official docs](https://dictionarry.dev/) · [v2 dev log](https://v2.dictionarry.dev/devlogs/profilarr-v2) · [AI transparency](https://v2.dictionarry.dev/ai-transparency)
- [Profilarr GitHub](https://github.com/Dictionarry-Hub/profilarr) · [Issues](https://github.com/Dictionarry-Hub/profilarr/issues) · [v2.x roadmap](https://github.com/Dictionarry-Hub/profilarr/milestone/2)
- [Discord](https://discord.gg/XGdTJP5G8a) · [r/Profilarr](https://www.reddit.com/r/Profilarr/)

# <img src="/youtube.png" class="tab-icon"> 17 · Video

https://youtu.be/PLACEHOLDER