---
title: Bazarr
description: A guide to deploying Bazarr via docker compose
published: true
date: 2025-07-11T12:31:21.875Z
tags: 
editor: markdown
dateCreated: 2024-09-01T11:48:39.955Z
---

# ![](/bazarr.png){class="tab-icon"} What is Bazarr?
Bazarr is a companion application to Sonarr and Radarr. It can manage and download subtitles based on your requirements. You define your preferences by TV show or movie and Bazarr takes care of everything for you.


# 1 · Deploy Bazarr
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](/screenshot_from_2024-11-08_11-40-44.png)

1. Change the **Timezone**. For east coast time, use **America/New\_York**.
1. Change the **Storage Configuration**. The **Bazarr Config Storage** *Type* should be set to **Host Path** as described in the [Folder-Structure](/Folder-Structure) guide.
1. Change the **WebUI Port** to **6767** as that is the default used in all the documentation. 
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/bazarr:/config
      - /mnt/tank/media:/media
    ports:
      - 6767:6767
    restart: unless-stopped
```

### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/bazarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

# 2 · Bazarr Configuration

The [wiki for Bazarr](https://wiki.bazarr.media/) is very good. Once you have the container running, follow the instructions [here](https://wiki.bazarr.media/Getting-Started/Setup-Guide/) and then [set the profile](https://wiki.bazarr.media/Getting-Started/First-time-installation-configuration/) for all of your existing content (if you have any). 

Before you do, you will probably want free account in these two places which will be used during the setup configuration:

1.  For **Providers** I used [opensubtitles.com](https://www.opensubtitles.com/en).
2.  For my anti-captcha I used [anti-captcha.com](https://anti-captcha.com/). ( ← If you stick to opensubtitles only, you won't need this but it is generally recommended to use more than one provider.)

# 3 · Video
[![](/2025-04-09-bazarr-automated-subtitle-manag-promo-card.png)](https://www.patreon.com/posts/bazarr-automated-126312662)