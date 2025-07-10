---
title: Tdarr
description: A guide to installing Tdarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-10T19:43:33.101Z
tags: 
editor: markdown
dateCreated: 2025-01-26T22:41:02.635Z
---

# ![](/tdarr.png){class="tab-icon"} What is Tdarr?

Tdarr is a tool that can help you optimize your media files by transcode, remux, remove unwanted streams and more. It supports cross-platform nodes, hardware transcoding, plugins and job reports.

# 1 · Deploy Tdarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  tdarr:
    container_name: tdarr
    image: ghcr.io/haveagitgat/tdarr:latest
    restart: unless-stopped
    network_mode: bridge
    ports:
      - 8265:8265 # webUI port
      - 8266:8266 # server port
    environment:
      - TZ=America/New_York
      - PUID=568
      - PGID=568
      - UMASK_SET=002
      - serverIP=0.0.0.0
      - serverPort=8266
      - webUIPort=8265
      - internalNode=true
      - inContainer=true
      - ffmpegVersion=6
      - nodeName=MyInternalNode
      - NVIDIA_DRIVER_CAPABILITIES=all
      - NVIDIA_VISIBLE_DEVICES=all
    volumes:
      - /mnt/tank/configs/tdarr/server:/app/server
      - /mnt/tank/configs/tdarr/configs:/app/configs
      - /mnt/tank/configs/tdarr/logs:/app/logs
      - /mnt/tank/configs/tdarr/transcode_cache:/temp
      - /mnt/tank/media:/media

    devices:
      - /dev/dri:/dev/dri
  #  deploy:
  #    resources:
  #      reservations:
  #        devices:
  #        - driver: nvidia
  #          count: all
  #          capabilities: [gpu]
```

### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/tdarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

> This compose is setup for an iGPU. Tdarr works best with a dGPU, and if you have an nVidia card, remove the 
> ```
> devices:  
>  - /dev/dri:/dev/dri
>  ``` 
> line and uncomment all the lines below it (starting with `deploy`). 
{.is-info}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](/screenshot_from_2025-01-26_17-42-01.png)

1. Under **Storage Configuration** use **Host Path** for each of the datasets (Tdarr Server Storage, Tdarr Configs Storage, Tdarr Logs Storage, Tdarr Transcode Storage) and point them to datasets you create in your pool. 
1. Click the **Add** button for **Additional Storage** to pass the `media` directory into the container.
1. At the very bottom, click the box to **Select NVIDIA GPU(s)** to pass your dGPU into the container. 

# 2 · Tdarr Configuration

## 2.1 Library (source tab)

![](/tdarrlib.png)

1.  Click the **Library** tab
2.  Click the **Library+** button
3.  Name your library (one or movies one for tv)
4.  Select the path inside the container (/media/movies or /media/tv)

## 2.2 Library (transcode cache tab)

![](/tdarr5.png)

10. Select the **Transcode Cache** tab

11. Enter the path `/temp`

## 2.3 Library (transcode options tab)

![](/tdarr2.png)

![](/tdarr3.png)

5. Click this box

6. Check the box to enable to GPU to transcode (if you have a dGPU)

![](/screenshot_from_2025-01-26_17-57-55.png)

Now scan the Library for new files.

## 2.4 Tdarr tab

![](/tdarr4.png)

7. Click the **Tdarr** tab

8. Click this box (yours will have a different label)

9. Increase your GPU workers

![](/screenshot_from_2025-01-26_18-01-29.png)

Scroll down to the **Staging Section** and click the box to **Auto accept successful transcodes**.

# 3 · Video

[![](/2025-01-26-truenas-scale--tdarr-efficient-promo-card.png)](https://www.patreon.com/posts/truenas-scale-120842366)