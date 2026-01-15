---
title: Install Instructions
description: A step-by-step list of the best way to install the *arr apps
published: true
date: 2026-01-15T15:27:33.400Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:31.090Z
---

# Steps

Follow the steps in this order to make the install go as smoothly as possible.

| Step | Instruction | Page |
| --- | --- | --- |
| 1 | Go through the TrueNAS page and set up your server following all of the steps outlined | [TrueNAS](/TrueNAS) |
| 2   | Set up the folder structure for the `/media` and `/configs` directory | [Folder Structure](/Folder-Structure) |
| 3   | Get your .conf file for your VPN | [AirVPN](/AirVPN) |
| 4   | Install qBittorrent | [qBittorrent](/qBittorrent) |
| 5   | Install Radarr | [Radarr](/radarr) |
| 6   | Install Sonarr | [Sonarr](/Sonarr) |
| 7   | Install Flaresolverr | [Flaresolverr](/Flaresolverr) |
| 8   | Install Prowlarr | [Prowlarr](/Prowlarr) |
| 9   | Install Recyclarr or Profilarr | [Recyclarr](/Recyclarr) / [Profilarr](/profilarr) |
| 10  | Install Emby/Jellyfin | [Emby](/Emby)/[Jellyfin](/jellyfin) |
| 11  | Install Unpackerr | [Unpackerr](/Unpackerr) |
| 12  | Install Jellyseerr | [Jellyseerr](/Jellyseerr) |
| 13  | Install Bazarr | [Bazarr](/bazarr) |
| 14  | Install Tdarr | [Tdarr](/tdarr) |
| 15  | If you want remote access, configure a VPN | [Cloudflare Tunnels](/CloudflareTunnels) / [wg-easy](/wg-easy) / [Tailscale](/tailscale) / [Netbird](/netbird) |
| 16  | If you don't have a static IP but want external access through a domain, add Cloudflare DDNS | [Cloudflare DDNS](/cloudflareddns) |
| 17  | If you want to monitor your apps, install Uptime Kuma | [Uptime Kuma](/Kuma) |
| 18  | Add notifications! | [Notifications](/Notifications) |

- I feel like this is the most logical order of progression. You can't really set up anything until your server is configured and your folders are set up.
 
- Once you are there, getting qBit up and running is probably the hardest thing to do because it has the most configuration. You also have to get AirVPN going first to get your .conf file before you can deploy qBit, so it makes sense to start with the VPN then move to the bittorrent client.
 
- After that Radarr and Sonarr will ask to be pointed to the media folders you already have set up as well as the download client which is now running.
 
- Install Flaresolverr so when we set up Prowlarr next, we can add it in case we will need it later. 
 
- Prowlarr will want to know which apps you want to connect, and Radarr and Sonarr are now ready to accept API pushes from Prowlarr, as well as the Flaresolverr server for indexers which use Cloudflare.
 
- Recyclarr or Profilarr will now need to set up quality profiles for Radarr and Sonarr, which are ready to accept them.
 
- Emby/Jellyfin can be deployed since our media folders are being managed by Radarr and Sonarr.
 
- Unpackerr can now be deployed since we have API keys for Radarr and Sonarr in the event we get .rar files.
 
- Jellyseerr will want to know which Radarr and Sonarr servers are set up and the default quality profile you want to use for downloads.

- If you want to sure up your subtitles, install Bazarr.

- Install Tdarr to compress H264 to save some hard drive space.

# Auto Install Script
I have created a script to create all the directories and install all the containers for you. [Go here](/Folder-Structure#auto-folder-creation-for-truenas) for more info. 

# <img src="/patreon-light.png" class="tab-icon"> Video

For a video walkthrough of some of the basics containers above, look here:

[![](/2025-01-28-truenas-scale-build-an-arr-stac-promo-card.png)](https://www.patreon.com/posts/truenas-scale-in-120976920)