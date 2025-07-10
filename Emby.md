---
title: Emby
description: A guide to installing Emby in TrueNAS as well as docker via compose
published: true
date: 2025-07-10T18:20:45.823Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:37:12.950Z
---

# ![](/emby2.png){class="tab-icon"} What is  Emby?

Emby is a personal media server that lets you access and enjoy your videos, music, and photos on any device. You can also stream live TV, manage your DVR, and control your content with parental controls and DLNA.

# 1 · Deploy Emby
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  emby:
    image: lscr.io/linuxserver/emby:latest
    container_name: emby
    runtime: nvidia # Expose NVIDIA GPUs
    network_mode: host # Enable DLNA and Wake-on-Lan
    environment:
      - PUID=568
      - PGID=568
      - NVIDIA_VISIBLE_DEVICES=all
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/emby:/config
      - /mnt/tank/media:/media
    ports:
      - 8096:8096 # HTTP port
      - 8920:8920 # HTTPS port

    restart: unless-stopped
```

### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/emby
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

> **Enabling nVidia GPUs in Docker**
> 
> Follow the steps [in this Github page](https://github.com/NVIDIA/nvidia-container-toolkit) to allow passing an nVidia GPU into a container
{.is-info}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](https://wiki.hydrology.cc/screenshot_from_2023-12-11_08-39-08.png)


1. Check the **Emby Server Storage** box to **Enable Host Path for Emby Server Config Volume** and select the correct path as described in the [Folder-Structure](/Folder-Structure) guide. 
1. Click the **Add** button for **Emby Server Extra Host Path Volumes** to pass the `media` directory into the container as **/media**.
1. Scroll to the **Resource Configuration** section and locate the checkbox for your GPU. If you do not see one, make sure to select the checkbox in **Apps → Configuration → Settings → Install NVIDIA Drivers**.

> Check out [the new docs](https://apps.truenas.com/resources/deploy-emby) from TrueNAS
{.is-success}


# 2 · Emby Configuration

## 2.1 Setup Media Libraries

1. Click **\+ New Library**
2. Set the Content Type to match the folder (Movies for /media/movies and TV Shows for /media/tv)
1. Click on the **+** next to Folders and add the appropriate folder
1. Click the green **OK** at the bottom and leave the rest of the settings the same
1. Do the same thing for your second library for TV Shows.

> **Spinning Blue Circle?**
> When you arrive at the home screen, you will probably see a spinning blue circle in the middle of the page. To stop this from happening in the future, click the **cog wheel** in the top right corner, then **Home Screen** in the left pane, and on **Section 4** change **Live TV** to **None**.
{.is-info}


## 2.2 Watching Content

To watch content on a device in your house on your home network, download the Emby App from wherever your device gets apps from. When you start the Emby app, manually add a server; don't click Emby Connect. Type in the IP Address of your server (for me it would look like http://192.168.1.215/) and use the default port of 8096. Once its connected you should see the sign-in screen with your name on it. If not, use the manual login and type the username and password you set up earlier.

# 3 · Using NPM
If you are using [Nginx Proxy Manager](/nginx) to route your traffic use these settings to be sure you're not caching media:

![screenshot_from_2025-03-28_07-39-15.png](/screenshot_from_2025-03-28_07-39-15.png)

| Field | Value |
| --- | --- |
| location | `/` |
| Cog Icon | click this to expand the box at the bottom |
| scheme | http |
| Forward Hostname / IP | private IPv4 of your media server |
| Forward Port | port media server is running on | 
| expandable box | `add_header Cache-Control "no-store, no-cache, must-revalidate, proxy-revalidate"; add_header Pragma "no-cache"; add_header Expires "0";` | 
