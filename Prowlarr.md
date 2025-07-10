---
title: Prowlarr
description: A guide to installing Prowlarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-10T18:23:52.667Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:31:22.190Z
---

# ![](/prowlarr.png){class="tab-icon"} What is Prowlarr?

Prowlarr is a indexer manager/proxy built on the popular arr .net/reactjs base stack to integrate with your various PVR apps. Prowlarr supports both Torrent Trackers and Usenet Indexers. It integrates seamlessly with Sonarr, Radarr, Lidarr, and Readarr offering complete management of your indexers with no per app Indexer setup required (we do it all).
# 1 · Deploy Prowlarr
# tabs {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/prowlarr:/config
      - /mnt/tank/media/:/media/
    ports:
      - 9696:9696
    restart: unless-stopped
```
### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/prowlarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](https://wiki.hydrology.cc/screenshot_from_2023-12-11_07-04-19.png)
- Install Prowlarr from the TrueNAS Community Apps catalog.
- Use the Community version when available.
- Change the **Config Storage Type** to **Host Path** as per the [Folder-Structure](/Folder-Structure) guide.
 - Click **Add** under **Additional Storage** to mount the media directory inside the container.

# 2 · Prowlarr Configuration

## 2.1 Indexers

This is where you will add public or private trackers you are a part of. In the top left corner you will see a "➕ Add Indexer" button. Click it and search for the name of the tracker you want to add. When you click the name of the tracker a form will open to enter the specific info of the tracker. 

## 2.2 Linking to Other \*arr's

In the menu on the left, navigate to **Settings** > **Apps** and click the ➕ icon. Select the app you would like to link. Leave all the options set to their defaults except the **Prowlarr Server** and **{App Name} Server**. The server line needs to be the IP and port Prowlarr and the  IP and port of the app you want to link. The **API Key** is copied from the app itself (in the app, navigate to **Settings** > **General** and copy the API Key from the Security section). Click **Test** then **Save**. 

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_14-28-39.png)

## 2.3 Download Client

Navigate in Prowlarr to **Settings** > **Download Client** and click the ➕ icon. Click the box for **qBittorrent**. Change the host to the IP of the server and the port to the correct port which qbit is running on. Use the credentials you set up when you installed qBit. Leave all other options as default and click **Test**, then **Save** at the bottom.

## 2.4 Flaresolverr

Usually this is only necessary if you use private trackers, but it won't hurt to set it up. Navigate to **Settings** > **Indexers** and then click the “**+**” box. Click the box for **Flaresolverr**. Enter **flaresolverr** for the **Tags** line and the IP of the server and use port 8191 for the **Host** line.

To apply Flaresolverr to an indexer which utilizes Cloudflare, when adding/editing the indexer, enter **flaresolverr** in the **Tags** line.

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_22-57-44.png)

## 2.5 Notifications

See [this section](https://wiki.serversatho.me/en/Notifications#radarrsonarrprowlarr) of the [Notifications](https://wiki.serversatho.me/Notifications) page.

## 2.6 General > Backups

1. Navigate to the bottom to the **Backups** section 
1. Change the folder to `/media`
1. Set the **Interval** to 1 and the **Retention** to your preference (default: 7 days).

# 3 · Prowlarr Behind a VPN

For those of you who are being blocked by your country's ISP, I recommend running Prowlarr in an LXC. The qbit LXC is already setup to do this. Follow the steps below to setup docker/dockge inside the LXC:
1. Go to [the qBit page](https://wiki.serversatho.me/en/qBittorrent#prerequisites) and install the LXC as described
1. [Run the script](https://wiki.serversatho.me/en/Docker#official-docker-script) from the instance shell to install docker in the LXC
1. [Install Dockge](/Dockge) and deploy Prowlarr like usual

# <img src="/patreon-light.png" class="tab-icon"> 4 · Video Walkthrough

![](/2025-01-30-complete-guide-to-prowlarr-the--promo-card.png)

[Watch it on Patreon!](https://www.patreon.com/posts/complete-guide-121131077?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)