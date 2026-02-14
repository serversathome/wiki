---
title: Jellyseerr
description: A guide to installing Jellyseerr in TrueNAS Scale as well as docker via compose
published: true
date: 2026-02-14T20:21:12.716Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:33.085Z
---

> **THIS CONTAINER HAS BEEN DEPRECATED IN FAVOR OF [Seerr](/seerr)**
{.is-danger}


# ![](/jellyseerr.png){class="tab-icon"} What is Jellyseerr?

Jellyseerr is a free and open source software application for managing requests for your media library. It is a a fork of Overseerr built to bring support for Jellyfin & Emby media servers!

# 1 · Deploy Jellyseerr
# tabs {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  jellyseerr:
    image: fallenbagel/jellyseerr:latest
    container_name: jellyseerr
    environment:
      - LOG_LEVEL=debug
      - TZ=America/New_York
    ports:
      - 5055:5055
    user: "568:568"
    volumes:
      - /mnt/tank/configs/jellyseerr:/app/config
    restart: unless-stopped
```

- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/sonarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.


## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](/screenshot_from_2024-02-23_09-37-02.png)

1. Change the **Timezone**. For east coast time, use **America/New\_York**.
1. Change the **Storage Configuration**. The **Jellyseerr Config Storage** *Type* should be set to **Host Path** as described in the [Folder-Structure](/Folder-Structure) guide.

# 2 · Jellyseerr Configuration

## 2.1 Signing In

1. Select **Sign in with Emby**
1. Enter the IP and Port for your Emby server and use your Emby username and password to sign in
1. The email you use doesn't matter at all - feel free to fake it
1. Click **Continue**.
1. Click the button to **Sync Libraries** and when you get the option, slide the button for both TV and Movies then click the button for **Start Scan**, then click **Continue**.

## 2.2 Adding our Apps

6. Click the area to **Add Radarr Server** (or Sonarr Server). Make the following changes to the pop-up:

|   Field  | Value    |
| --- | --- |
| **Default Server** | check this box if you are only installing one instance of Radarr/Sonarr |
| **4K Server** | this is for advanced users who run two different servers for 4K and 1080p content |
| **Server Name** | usually call it either Radarr or Sonarr |
| **Host or IP Address** | IP of the Radarr or Sonarr server |
| **Port** | port of the Radarr or Sonarr server |
| **Use SSL** | leave this unchecked |
| **API Key** | API Key for Radarr or Sonarr |

7. After you populate these values, click **Test** at the bottom. Now you should be able to enter the rest of the values on the table.

|   Field  |   Value  |
| --- | --- |
| **URL Base** | leave this blank |
| **Quality Profile** | select the quality profile content will be downloaded with by default |
| **Root Folder** | there should only be one option here; select the folder you set up for Radarr/Sonarr |
| **Minimum Availability** | change this to **Announced** |
| **Tags** | leave blank |
| **External URL** | leave blank |
| **Enable Scan** | check this box |
| **Enable Automatic Search** | check this box |

8. Now you can select **Add Server** at the bottom. Do the same for Sonarr. 

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_14-36-31.png)

# <img src="/patreon-light.png" class="tab-icon"> 3 · Video Walkthrough

[![](/2025-02-10-the-complete-guide-to-jellyseerr-promo-card.png)](https://www.patreon.com/posts/complete-guide-121946119?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)