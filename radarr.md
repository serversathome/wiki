---
title: Radarr
description: A guide to installing Radarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-09T12:06:16.535Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:11.647Z
---

# ![](/radarr.png){class="tab-icon"} What is Radarr?

Radarr is a movie collection manager for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new movies and will interface with clients and indexers to grab, sort, and rename them. It can also be configured to automatically upgrade the quality of existing files in the library when a better quality format becomes available.

# 1 · Deploy Radarr

Radarr can be installed using Docker Compose or directly via the TrueNAS SCALE Community Apps Catalog.

# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/radarr:/config
      - /mnt/tank/media:/media
    ports:
      - 7878:7878
    restart: unless-stopped
```
### Permissions & Folder Structure

- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/radarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

## <img src="/truenas.png" class="tab-icon"> TrueNAS

![I use the Community version of all apps when available](https://wiki.hydrology.cc/screen_shot_2023-12-08_at_1.38.00_pm.png) 

- Install Radarr from the TrueNAS Community Apps catalog.
- Use the Community version when available.
- Change the **Config Storage Type** to **Host Path** as per the [Folder-Structure](/Folder-Structure) guide.
 - Click **Add** under **Additional Storage** to mount the media directory inside the container.

# 2 · Radarr Configuration

## 2.1 Root Folder

1. Navigate in Radarr to **Settings** > **Media Management**
1. click the button to **Add Root Folder**. 
1. Select the folder path to `/media/movies`.

## 2.2 Download Client

1. Navigate in Radarr to **Settings** > **Download Client**
1. Click the ➕ icon. Click the box for **qBittorrent**. 
1. Configure the following:

| **Setting** | **Value** |
|----------|----------|
| Host      | Server IP      |
| Port     | qBittorrent WebUI port    |
|Use Credentials |Set up during qBittorrent installation  |

4. Click **Test**, then **Save**.

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_14-31-16.png)

# 3 ·  Advanced Settings

The below settings are for advanced users and are totally optional. To see all of the Advanced fields (usually colored orange) be sure to click the cog icon in the top left of the window to **Show Advanced**. 

>This also assumes you are using Recyclarr to sync data to Radarr. If you are not, do not continue. See the video walkthrough below for more details.
{.is-warning}

## 3.1 Media Management

| **Field** | **Value** |
| --- | --- |
| Rename Movies | True |
| Standard Movie Format | {Movie CleanTitle} {(Release Year)} \[imdbid-{ImdbId}\] - {Edition Tags }{\[Custom Formats\]}{\[Quality Full\]}{\[MediaInfo 3D\]}{\[MediaInfo VideoDynamicRangeType\]}{\[Mediainfo AudioCodec}{ Mediainfo AudioChannels\]}{\[Mediainfo VideoCodec\]} |
| Propers and Repacks | Do Not Prefer |
| Recycling Bin Cleanup | 1   |
| Set Permissions | True |
| chmod Folder | 777 |

## 3.2 Profiles

- Delete all profiles not created by Recyclarr.
- Change the default profile for Jellyseerr to the new ones created by Recyclarr. 
- If you cannot delete a profile it is because it is in use by a movie. Reassign the profile before deleting the old one.

> **Why use Recyclarr profiles?** 
Recyclarr automates profile creation and ensures consistent quality filtering, metadata tagging, and download prioritization for a better Radarr experience.
{.is-info}


## 3.3 Connect

See [this section](/en/Notifications#radarrsonarrprowlarr) of the [Notifications](/Notifications) page.

## 3.4 Metadata

Select **Kodi (XBMC)/Emby** and check the **Enable** box. Save and close.
> **Why enable metadata?**
Enabling metadata allows Radarr to send detailed episode information, artwork, and tags to your media server (Kodi/Emby), enhancing library organization.
{.is-info}


## 3.5 General > Backups

1. Scroll all the way to the bottom to the **Backups** section
1. Change the folder to `/media`
1. Set the **Interval** to 1 
1. Set the **Retention** to your preference. (default: 7 days)

# 4 · Troubleshooting
## 4.1 Radarr Cannot See Media Files

1. Ensure your volume paths in Docker match the root folder in Radarr.
1. Run to check file ownership and permissions:
```bash
ls -lah /mnt/tank/media/movies
```
3. If needed, change ownership using:
```bash
chown -R 568:568 /mnt/tank/media/movies
```
## 4.2 Permission Errors (Permission Denied)
1. Ensure your user (PUID / PGID) has access to the media folders.
2. Try running:
```bash
chmod -R 770 /mnt/tank/media/movies
```
## 4.3 Radarr is Downloading But Not Moving Files

- Verify that your Download Client Path Mapping is correctly set.
- In Settings > Download Clients, ensure Radarr has access to the completed download folder.

# Video Walkthrough

[![](/2025-03-18-advanced-media-management-with-r-promo-card.png)](hhttps://www.patreon.com/posts/advanced-media-124637606)