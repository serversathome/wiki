---
title: Profilarr
description: A guide to deploying Profilarr with docker compose
published: true
date: 2026-05-23T12:05:05.419Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:30.974Z
---

# <img src="/profilarr.png" class="tab-icon"> What is Profilarr V2?
Configuration management tool for Radarr/Sonarr that automates importing and version control of custom formats and quality profiles.

🔄 Automatic synchronization with remote configuration databases
🎯 Direct import to Radarr/Sonarr instances
🔧 Git-based version control of your configurations
⚡ Preserve local customizations during updates
🛠️ Built-in conflict resolution

>  v2 is not compatible with v1. The underlying database and customization systems changed significantly, so existing v1 databases/configs/appdata won't work directly in v2
{.is-danger}
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Profilarr V2
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
```

# 2 · Configuration
> The official guide from Profilarr is excellent. I recommend you read it [here](https://dictionarry.dev/)
{.is-info}

## 2.1 Initial Setup
1. When logging in the first time set a username and password
1. Navigate to **Databases** and click the "**+**" button to **Link Repository**. Dictionary-Hub repo is added by default.
	a. Enter the name of the database
  b. The repository URL
  c. Enter the branch (Main, Dev, etc.) *optional*
  d. Enter a "Personal Access Token" if required
1. Select **Save** at the top right of the page. 

> A list of other repos can be found [here](https://github.com/Dictionarry-Hub/database/forks?include=active&page=1&period=&sort_by=stargazer_counts_)
{.is-info}

>   Currently Profilarr does not sync directly with Trash Guides
{.is-warning}

> If you are downloading anime, add the repo from the [Dumpstarr repo](https://github.com/Dumpstarr/Database): `https://github.com/Dumpstarr/Database`
{.is-success}

> The `serversathome/profilarr` repo is now deprecated and **will be removed** before July 1, 2026
{.is-danger}




# 3 · Arrs: Adding Radarr/Sonarr
1. Navigate to **Arrs** on the left side of the screen.
1. Click **Add Instance**
1. Enter Name
1. Enter URL
1. Enter API Key
1. Set **Library Refresh** to `Every 24 hours`
1. Enable **Cleanup** on a `Weekly` schedule
1. Click save at the top right of the page.


## 3.1 Sync
### 3.1.1 Media Management
1. Set **Naming**, **Quality Definitions**, and **Media Settings** to your database of choice
1. Set **Trigger** to `on pull`
1. **Save** and then **Sync**

### 3.1.2 Delay Profiles
1. Select your delay profile
1. Set **Trigger** to `on pull`
1. **Save** and then **Sync**

### 3.1.3 Quality Profiles
1. Select your desired **Quality Profiles**
1. Set **Trigger** to `on pull`
1. **Save** and then **Sync**

## 3.2 Drift
1. Set **Detection** to `Enabled`
1. Set **Schedule** to `Daily`
1. **Save** and then **Run Now**

## 3.3 Upgrades
**Upgrades** is the replacement to the functionality that *Huntarr* used to provide, so use it the same way and do not have two different services doing upgrades simultaneously. Note that this feature is optional if you already use a replacement service which you like.

1. Set **Status** to `Enabled`
1. Set **Schedule** to `Hourly`
1. Click <kbd>+</kbd> at the end of the **Search Filters** row to add the **Default** filter
1. Click **Save**
1. Optionally increase your **Count** or the **Match Filters** if you want to get more specific, but the defaults work well.


## 3.4 Renames
1. Set **Status** to `Enabled`
1. Set **Schedule** to `Daily`
1. **Save** and then **Run Now**





# 4 · Media Management
Profilarr allows us to change the defaults for other catagories in Radarr/Sonarr on a per repo basis:

- Naming Settings: Naming file/folder, Character replacement
- Quality Definitions: Adjust quality scoring
- Media Settings: Rename config, set Propers and Repacks, enable media info scan

The defaults are good and do not need to be adjusted but are exposed for users with specific use-cases.


# 5 · Delay Profiles
1. For Radarr/Sonarr lower your **Delay** to a wait time your prefer (I use 180 minutes)
1. In the **Bypass Conditions** enable both *Bypass if Highest Quality* and *Bypass if Above Custom Format Score (0)*



# <img src="/patreon-light.png" class="tab-icon"> 6 · Video

