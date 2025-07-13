---
title: Navidrome
description: A guide to deploying the Navidrome music player using docker
published: true
date: 2025-07-13T11:02:08.748Z
tags: 
editor: markdown
dateCreated: 2025-07-13T10:43:27.715Z
---

# ![](/navidrome.png){class="tab-icon"}What is Navidrome?
Navidrome can be used as a standalone server, that allows you to browse and listen to your music collection using a web browser.


# <img src="/docker.png" class="tab-icon"> Deploy Navidrome

```yaml
services:
  #----------- MUSIC PLAYER -----------#
  navidrome:
    image: deluan/navidrome:latest
    user: 568:568
    ports:
      - 4533:4533
    restart: unless-stopped
    environment:
      ND_SCANNER_SCHEDULE: 1h # Optional
      ND_LOGLEVEL: info # Optional
      ND_SESSIONTIMEOUT: 12h # Optional
      #ND_BASEURL: ""
    volumes:
      - /mnt/tank/configs/navidrome/data:/data
      - /mnt/tank/media/music:/music
```

## Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/emby
    - **Dockge**: you can pass the dockge ./config/ folder structure if you are using that instead
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

> Refer to the offical [Navidrome docker installation steps](https://www.navidrome.org/docs/installation/docker/) for further information on the optional environment variables. 
{.is-success}


That's it, there isn't much more to it but to ensure your music files are placed in the appropriate mounted path.

For music management and tagging, many users recommend using [MusicBrainz Picard](https://picard.musicbrainz.org/) or [Beets](https://beets.io/), which can ensure that your Navidrome library is as clean and well-organized as possible.