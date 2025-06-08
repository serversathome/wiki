---
title: Profilarr
description: A guide to deploying Profilarr with docker compose
published: true
date: 2025-06-08T18:40:16.703Z
tags: 
editor: markdown
dateCreated: 2025-03-23T11:41:17.078Z
---

![screenshot_from_2025-03-23_08-05-46.png](/screenshot_from_2025-03-23_08-05-46.png)
[See it on GitHub](https://github.com/Dictionarry-Hub/profilarr)

# What is Profilarr?
Configuration management tool for Radarr/Sonarr that automates importing and version control of custom formats and quality profiles.

🔄 Automatic synchronization with remote configuration databases
🎯 Direct import to Radarr/Sonarr instances
🔧 Git-based version control of your configurations
⚡ Preserve local customizations during updates
🛠️ Built-in conflict resolution


# Installation
```yaml
services:
  profilarr:
    image: santiagosayshey/profilarr:latest
    container_name: profilarr
    ports:
      - 6868:6868
    volumes:
      - /mnt/tank/configs/profilarr:/config
    environment:
      - TZ=America/New_York
    restart: unless-stopped
```

# Configuration
> The official guide from Profilarr is excellent. I recommend you read it [here](https://dictionarry.dev/wiki/profilarr-setup)
{.is-info}

## Initial Setup
1. When logging in the first time set a username and password
1. Navigate to **Settings** → **Database** and click the button to **Link Repository**
	a. Enter `https://github.com/Dictionarry-Hub/database`
> A list of other repos can be found [here](https://github.com/Dictionarry-Hub/database/forks?include=active&page=1&period=&sort_by=stargazer_counts_)
{.is-info}

>   Currently Profilarr does not sync directly with Trash Guides
{.is-warning}

3. Move to slider to **Enable Auto Sync**

## Adding Radarr/Sonarr
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


## Permanently Modifying Profiles
> Any changes you make to the auto-populated definitions will not persist unless you make a commit
{.is-warning}
1. Once your changes are made navigate to **Settings** → **Database** and click the action button to select your modifications
1. Click the ➕ button to Stage your changes
1. Enter the information about your change
1. Click the **Commit** button

# Future Development
This software is *very* new. As such, there are some changes coming that will improve its usage: 
- The ability to add multiple repos at the same time
- A setting to overwrite external changes with local changes when using auto-pull
- A more fluid way to commit changes with less clicks
- Profilarr will handle a quick setup sync (changing media management, quality slider settings, etc)

# Video Walkthrough
[](https://youtu.be/u1FQNMsuzFc)

[^1]: Automatically syncs selected files whenever the database receives an update. When combined with Auto Pull, allows Profilarr to work completely autonomously