---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-06-08T18:39:06.576Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

![](https://wiki.hydrology.cc/sonarr.png)

# What is Sonarr?

Sonarr is a PVR for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite shows and will grab, sort and rename them. It can also be configured to automatically upgrade the quality of files already downloaded when a better quality format becomes available.

# Installation
# tabs {.tabset}
## Docker Compose

```yaml
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/sonarr:/config
      - /mnt/tank/media:/media
    ports:
      - 8989:8989
    restart: unless-stopped
```
### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/sonarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

## TrueNAS

![](/screen_shot_2023-12-08_at_3.04.39_pm.png)

- Install Sonarr from the TrueNAS Community Apps catalog.
- Use the Community version when available.
- Change the **Config Storage Type** to **Host Path** as per the [Folder-Structure](/Folder-Structure) guide.
 - Click **Add** under **Additional Storage** to mount the media directory inside the container.
 
# Sonarr Configuration

## Root Folder

- Navigate to **Settings** > **Media Management**
- Click the button to **Add Root Folder**
- Select the `/media/tv` directory

## Download Client

1. Navigate in Sonarr to **Settings** > **Download Client**
1. Click the ➕ icon. Click the box for **qBittorrent**
1. Configure the following:

| Setting | Value |
| --- | --- |
| Host | Server IP |
| Port | qBittorrent WebUI port |
| Credentials | Set up during qBittorrent installation |

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_14-33-11.png)

# Advanced Settings

The below settings are for advanced users and are totally optional. To see all of the Advanced fields (usually colored orange) be sure to click the cog icon in the top left of the window to **Show Advanced**. 

> This assumes you are using Recyclarr to sync data to Sonarr. If you are not, do not continue. See the video walkthrough below for more details. 
{.is-warning}


## Media Management

| **Field** | **Value** |
| --- | --- |
| Rename Episodes | True |
| Standard Episode Format | {Series TitleYear} - S{season:00}E{episode:00} - {Episode CleanTitle} \[{Custom Formats }{Quality Full}\]{\[MediaInfo VideoDynamicRangeType\]}{\[Mediainfo AudioCodec}{ Mediainfo AudioChannels\]}{\[MediaInfo VideoCodec\]}{-Release Group} |
| Daily Episode Format | {Series TitleYear} - {Air-Date} - {Episode CleanTitle} \[{Custom Formats }{Quality Full}\]{\[MediaInfo VideoDynamicRangeType\]}{\[Mediainfo AudioCodec}{ Mediainfo AudioChannels\]}{\[MediaInfo VideoCodec\]}{-Release Group} |
| Anime Episode Format | {Series TitleYear} - S{season:00}E{episode:00} - {absolute:000} - {Episode CleanTitle} \[{Custom Formats }{Quality Full}\]{\[MediaInfo VideoDynamicRangeType\]}\[{MediaInfo VideoBitDepth}bit\]{\[MediaInfo VideoCodec\]}\[{Mediainfo AudioCodec} { Mediainfo AudioChannels}\]{MediaInfo AudioLanguages}{-Release Group} |
| Series Folder Format | {Series TitleYear} \[imdbid-{ImdbId}\] |
| Episode Title Required | Never |
| Propers and Repacks | Do Not Prefer |
| Recycling Bin Cleanup | 1   |
| Set Permissions | True |
| chmod Folder | 777 |

## Profiles

1. Delete all profiles not created by Recyclarr
1. Change the default profile for Jellyseerr to the new ones created by Recyclarr
1. If you cannot delete a profile it is because it is in use by a series. Find out which series uses it and change it to the new profile, then go back and delete the old one.

> **Why use Recyclarr profiles?**
>Recyclarr automates profile creation and ensures consistent quality filtering, metadata tagging, and download prioritization for a better Sonarr experience.
{.is-info}


## Connect (Notifications)

See [this section](https://wiki.serversatho.me/en/Notifications#radarrsonarrprowlarr) of the [Notifications](/Notifications) page.

## Metadata

Select **Kodi (XBMC)/Emby** and check the **Enable** box. Save and close.
> **Why enable metadata?**
> Enabling metadata allows Sonarr to send detailed episode information, artwork, and tags to your media server (Kodi/Emby), enhancing library organization.
{.is-info}


## General > Backups

1. Navigate to the bottom to the **Backups** section 
1. Change the folder to `/media`
1. Set the **Interval** to 1 and the **Retention** to your preference (default: 7 days).

# Troubleshooting
## Sonarr Cannot See Media Files

- Ensure your volume paths in Docker match the root folder in Sonarr.
- Run to check file ownership and permissions:
``` bash
ls -lah /mnt/tank/media/tv
```
- If needed, change ownership using:
``` bash
chown -R 568:568 /mnt/tank/media/tv
```

## Permission Errors (Permission Denied)

Ensure your user (PUID / PGID) has access to the media folders.
Try running:
``` bash
chmod -R 770 /mnt/tank/media/tv
```

## Sonarr is Downloading But Not Moving Files
- Verify that your Download Client Path Mapping is correctly set.
- In **Settings** > **Download Clients** ensure Sonarr has access to the completed download folder.


# Video

![2025-03-24-advanced-media-management-with-s-promo-card.png](/2025-03-24-advanced-media-management-with-s-promo-card.png)
[Watch it on Patreon!](https://www.patreon.com/posts/advanced-media-124639393?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)
