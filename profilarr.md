---
title: Profilarr
description: A guide to deploying Profilarr with docker compose
published: true
date: 2026-01-11T09:04:20.240Z
tags: 
editor: markdown
dateCreated: 2025-03-23T11:41:17.078Z
---

# <img src="/profilarr.png" class="tab-icon"> What is Profilarr?
Configuration management tool for Radarr/Sonarr that automates importing and version control of custom formats and quality profiles.

🔄 Automatic synchronization with remote configuration databases
🎯 Direct import to Radarr/Sonarr instances
🔧 Git-based version control of your configurations
⚡ Preserve local customizations during updates
🛠️ Built-in conflict resolution


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Profilarr
```yaml
services:
  profilarr:
    image: santiagosayshey/profilarr:beta
    container_name: profilarr
    ports:
      - 6868:6868
    volumes:
      - /mnt/tank/configs/profilarr:/config
    environment:
      - TZ=America/New_York
    restart: unless-stopped
```

# 2 · Configuration
> The official guide from Profilarr is excellent. I recommend you read it [here](https://dictionarry.dev/)
{.is-info}

## 2.1 Initial Setup
1. When logging in the first time set a username and password
1. Navigate to **Settings** → **Database** and click the button to **Link Repository**
	a. Enter `https://github.com/Dictionarry-Hub/database`
> A list of other repos can be found [here](https://github.com/Dictionarry-Hub/database/forks?include=active&page=1&period=&sort_by=stargazer_counts_)
{.is-info}

>   Currently Profilarr does not sync directly with Trash Guides
{.is-warning}

> If you are downloading anime, use the repo from the [Anime Section](https://wiki.serversatho.me/en/profilarr#h-4-anime) below
{.is-success}


3. Move to slider to **Enable Auto Sync**

## 2.2 Adding Radarr/Sonarr
1. Navigate to **Settings** → **External Apps**
1. Click **Add**
![screenshot_from_2025-03-23_17-18-36.png](/screenshot_from_2025-03-23_17-18-36.png)

| Field | Value |
| --- | --- |
| Name | Name your instance |
| Type | Select type as Radarr or Sonarr |
| Arr Server | Enter the IP of your server using http://|
| API Key | Paste the API key from **Settings** → **General**| 
| Sync Method | Set sync to **On Pull** for full automation [^1]|
| Import as Unique | Use if you have quality profiles from another place like Trash Guides |

> Before sycing your apps change **Propers and Repacks** to **Do not Prefer** in  *advanced* **Settings** → **Media Management** in Radarr in Sonarr
{.is-danger}


## 2.3 Permanently Modifying Profiles
> Any changes you make to the auto-populated definitions will not persist unless you make a commit
{.is-warning}
1. Once your changes are made navigate to **Settings** → **Database** and click the action button to select your modifications
1. Click the ➕ button to Stage your changes
1. Enter the information about your change
1. Click the **Commit** button

# 3 · Media Management

There are some other tweaks which need to be done in Sonarr and Radarr. You can find these in the Advanced section of their respective pages, however, Profilarr can automate that for us.

1. Navigate to the **Media Management** tab
1. Click the **Radarr** subtab
1. Click the **Sync All** button in the top right
1. Repeat this process for Sonarr

# 4 · Anime

To get an anime profile you could either build one yourself through the UI or link to the Servers@Home Profilarr repo. To link, instead of using the Dictionarry link from step 2.1.2.a, use the link below:
```bash
https://github.com/serversathome/profilarr
```

# 5 · Future Development
This software is *very* new. As such, there are some changes coming that will improve its usage: 
- The ability to add multiple repos at the same time
- A setting to overwrite external changes with local changes when using auto-pull
- A more fluid way to commit changes with less clicks

# <img src="/youtube.png" class="tab-icon"> 6 · Video
[](https://youtu.be/u1FQNMsuzFc)

[^1]: Automatically syncs selected files whenever the database receives an update. When combined with Auto Pull, allows Profilarr to work completely autonomously