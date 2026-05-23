---
title: Profilarr
description: A guide to deploying Profilarr with docker compose
published: true
date: 2026-05-23T10:44:45.180Z
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
2. Navigate to **Databases** and click the "**+**" button to **Link Repository**. Dictionary-Hub repo is added by default.
	a. Enter the name of the database
  b. The repository URL. {**The database must include a pcd.json file.}**
  c. Enter the branch if you know it. "Main, Dev, etc."     * *Not a requirement*
  d. Enter a "Personal Access Token" if required.     * *Not a requirement*
![profilarr-v2-databases.png](/profilarr-v2-databases.png)
3. Select **Save** at the top right of the page. 
> A list of other repos can be found [here](https://github.com/Dictionarry-Hub/database/forks?include=active&page=1&period=&sort_by=stargazer_counts_)
{.is-info}

>   Currently Profilarr does not sync directly with Trash Guides
{.is-warning}

> If you are downloading anime, use the repo from the [Dumpstarr repo](https://github.com/Dumpstarr/Database).
{.is-success}

> The serversathome/profilarr repo will be archived. Recommend updating to V2 and use recommended anime repo.
{.is-danger}




# 3 ARRS - Adding Radarr/Sonarr
1. Navigate to **Arrs** on the left side of the screen.
2. Click **Add Instance**
![profilarr-v2-arrs.png](/profilarr-v2-arrs.png)

| Field | Value |
| --- | --- |
| Name | Name your instance |
| Type | Select type as Radarr or Sonarr |
| Arr Server | Enter the IP of your server using http://|
| API Key | Paste the API key from **Settings** → **General**| 
| Tags | Connects Profilarr to your Radarr and Sonarr instances. [^1]|
3. Test your connection.
4. Click save at the top right of the page.


## 3.1 Radarr/Sonarr Settings
> Any changes you make will not persist unless you Save the selection
{.is-warning}
1. Sync: Media management settings, Delay Profiles, Quality Profiles [^2]
		* Trigger - for how you want to sync to your Radarr/Sonarr "*Manual, On Pull, Schedule*"
2. Library: List of Movies/TV Shows for that app
3. Drift: Shows differences between app to profilarr
4. Upgrades: Upgrade on a schedule
5. Renames: Rename files/folders
> Suggest doing a dry run, may have unforeseen actions.
{.is-warning}

6. Logs: Shows for each arr application actions taken


# 4 · Extras
1. Quality Profiles: Quality definitions and scoring.
2. Custom Formats: Specifies release characteristics then add scoring.
3. Regular Expressions: Definitions for formats added to a file structure such as "**amzn**" for **Amazon Prime** as a streaming service tag for scoring. "Formerly known as Regex Patterns in V1"
> Can adjust for each repo added
{.is-info}


# 4 · Media Management
There are some other tweaks which need to be done in Sonarr and Radarr. You can find these in the Advanced section of their respective pages, however, Profilarr can automate that for us.
1. Naming Settings: Naming file/folder, Character replacement
2. Quality Definitions: Adjust quality scoring
3. Media Settings: Rename config, set Propers and Repacks, enable media info scan
> Can adjust for each repo added
{.is-info}

> Before sycing your apps change Propers and Repacks to Do not Prefer
{.is-danger}


# 5 · Anime

To get an anime profile you could either build one yourself through the UI or link to the Dumpstarr repo. To link, instead of using the Dictionarry link from step 2.1, use the link below:
```bash
https://github.com/Dumpstarr/Database
```
> Only one repo can be used per arr. It is recommended to separate your anime into a different instances.
{.is-warning}

# 6 · Future Development
This software is *very* new. As such, there are some changes coming that will improve its usage: 
[Road Map](https://github.com/Dictionarry-Hub/profilarr/milestone/2)

# <img src="/patreon-light.png" class="tab-icon"> 6 · Video



[^1]: Leave blank unless you are using tags already. If using tags, assign here for instance bridging.
[^2]: Every selection will need to be filled.