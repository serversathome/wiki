---
title: Cross Seed
description: A guide on how to deploy Cross Seed
published: true
date: 2025-06-11T11:53:18.218Z
tags: 
editor: markdown
dateCreated: 2025-06-11T09:31:37.247Z
---

![cross-seed.png](/cross-seed.png)


# What is Cross Seed?
cross-seed is an app designed to help you download torrents that you can cross seed based on your existing torrents. It is designed to match conservatively to minimize manual intervention. cross-seed can inject the torrents it finds directly into your torrent client. 

> This container requires the ability to hardlink to your media files. If you have not read the [Folder Structure](/Folder-Structure) Guide, I recommend you have you folders set up as-described so Cross Seed can function properly.
{.is-warning}


# Docker Compose
```yaml
services:
  cross-seed:
    image: ghcr.io/cross-seed/cross-seed:6
    container_name: cross-seed
    user: 568:568
    ports:
      - "2468:2468"
    volumes:
      - /mnt/tank/configs/crossseed:/config
      - /mnt/tank/media:/media
    command: daemon
    restart: unless-stopped
```

# Configuration
1. Navigate to the `configs` folder and open the `config.js` file
1. E
1. Start the daemon by running this command in the TrueNAS Shell as `root`:
```bash
docker exec -it cross-seed cross-seed daemon
```

# Adding qBit Scripts
> 
> This assumes Cross Seed and qBit are within the same Docker network and can reach eachother using the container name (eg http://qbittorrent:8080)
{.is-warning}

Cross Seed has the ability upon completion of a download to automatically push the torrent to other indexers instead of waiting for the scan at a later time to take advantage of earlier, larger leeching. To activate this feature follow the steps below:
1. Get the API key for cross seed by running the command below in the TrueNAS shell:
```bash
docker exec -it cross-seed cross-seed api-key
```
1. In qBit, naviagte to **Tools → Options → Downloads**
1. Enable **Run external program on torrent completion**, replacing `<API_KEY>` with the correct values from above and use this command:
```bash
curl -XPOST http://cross-seed:2468/api/webhook?apikey=<API_KEY> -d "infoHash=%I"
```

# Deleting Media
Since Cross Seed will create hardlinks in your directories, to remove unwanted media you need to remove it both from Sonarr/Radarr **as well as** your download client or the media will remain on your hard drive.

# Troubleshooting
The Cross Seed docs are excellent. Follow their instructions [here](https://www.cross-seed.org/docs/basics/faq-troubleshooting).

# Video