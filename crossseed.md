---
title: Cross Seed
description: A guide on how to deploy Cross Seed
published: true
date: 2026-01-15T15:28:47.987Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:08.228Z
---

# ![](/cross-seed.png){class="tab-icon"} What is Cross-Seed?
Cross-seed is an app designed to help you download torrents that you can cross seed based on your existing torrents. It is designed to match conservatively to minimize manual intervention. cross-seed can inject the torrents it finds directly into your torrent client. 

> This container requires the ability to hardlink to your media files. If you have not read the [Folder Structure](/Folder-Structure) Guide, I recommend you have you folders set up as-described so Cross Seed can function properly.
{.is-warning}

# ## <img src="/docker.png" class="tab-icon"> 1 · Deploy Cross-Seed
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

# 2 · Configuration

> This assumes Cross Seed and the other containers are within the same Docker network and can reach each other using the container name (eg http://radarr:8989)
{.is-warning}

1. Use the command below in the TrueNAS shell to open the `config.js` file for editing:
    ```bash
    nano /mnt/tank/configs/crossseed/config.js
    ``` 
    <details><summary>Instead of editing the lines one by one, you could simply copy and paste my preconfigured config.js file</summary>

    ```js
    "use strict";
    module.exports = {
        apiKey: undefined,

        torznab: [
            "http://prowlarr:9696/1/api?apikey=12345",
            "http://prowlarr:9696/2/api?apikey=12345",
        ],

        sonarr: ["http://sonarr:8989/?apikey=12345"],

        radarr: ["http://radarr:7878/?apikey=12345"],

        host: "0.0.0.0",
        port: 2468,

        notificationWebhookUrls: [],

        torrentClients: ["qbittorrent:http://user:pass@qbittorrent:8080"],

        useClientTorrents: true,

        delay: 30,

        dataDirs: ["/media/movies", "/media/tv"],

        linkCategory: "cross-seed-link",

        linkDirs: ["/media/downloads"],

        linkType: "hardlink",

        flatLinking: false,

        matchMode: "flexible",

        skipRecheck: true,

        autoResumeMaxDownload: 52428800,

        ignoreNonRelevantFilesToResume: false,

        maxDataDepth: 4,

        torrentDir: null,

        outputDir: null,

        includeSingleEpisodes: false,

        includeNonVideos: false,

        seasonFromEpisodes: 1,

        fuzzySizeThreshold: 0.02,

        excludeOlder: "2 weeks",
  
        excludeRecentSearch: "3 days",

        action: "inject",

        duplicateCategories: false,

        rssCadence: "30 minutes",

        searchCadence: "1 day",

        snatchTimeout: "30 seconds",

        searchTimeout: "2 minutes",

        searchLimit: 400,

        blockList: [],
    };
   //# sourceMappingURL=config.template.cjs.map
   ```
    </details>
   
   
1. Edit the `torznab` section and add your Prowlarr info. Get the number before the API key by clicking the indexer name in Prowlarr and looking at under the **Indexer Details** header.
    ```json
        torznab: [
            "http://prowlarr:9696/1/api?apikey=12345",
            "http://prowlarr:9696/2/api?apikey=12345",
        ],
    ```
1. Edit the `torrentClients` section and add your qbittorrent info
    ```json
        torrentClients: [
            "qbittorrent:http://user:pass@qbittorrent:8080",
        ],
    ```
1. Edit the `linkDirs` section and use the path `/media/downloads`
    ```json
        linkDirs: ["/media/downloads"],
    ```
1. Edit the `dataDirs` section and make it look like this: 
    ```json
    dataDirs: ["/media/movies", "/media/tv"]
    ```
1. Edit the Radarr/Sonarr section and add your info:
    ```json
      sonarr: ["http://localhost:8989/?apikey=12345"],
      radarr: ["http://localhost:7878/?apikey=67890"],
    ```

1. Restart the container

# 3 · Adding qBit Scripts

> This assumes Cross Seed and qBit are within the same Docker network and can reach each other using the container name (eg http://qbittorrent:8080)
{.is-warning}

> I have not gotten this to work because qBit is behind a VPN and cannot reach other containers
{.is-danger}


Cross Seed has the ability upon completion of a download to automatically push the torrent to other indexers instead of waiting for the scan at a later time to take advantage of earlier, larger leeching. To activate this feature follow the steps below:
1. Get the API key for cross seed by running the command below in the TrueNAS shell as `root`:
    ```bash
    docker exec -it cross-seed cross-seed api-key
    ```
1. In qBit, naviagte to **Tools → Options → Downloads**
1. Enable **Run external program on torrent completion**, replacing `<API_KEY>` with the correct values from above and use this command:
    ```bash
    curl -XPOST http://cross-seed:2468/api/webhook?apikey=<API_KEY> -d "infoHash=%I"
    ```

# 4 · Deleting Media
Since Cross Seed will create hardlinks in your directories, to remove unwanted media you need to remove it both from Sonarr/Radarr **as well as** your download client or the media will remain on your hard drive.

# 5 · Troubleshooting
The Cross Seed docs are excellent. Follow their instructions [here](https://www.cross-seed.org/docs/basics/faq-troubleshooting).

# <img src="/patreon-light.png" class="tab-icon"> 6 · Video

[![](/2025-06-11-how-to-set-up-cross-seed-power--promo-card.png)](https://www.patreon.com/posts/how-to-set-up-up-131256415)