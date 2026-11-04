---
title: FAQ
description: A list of simple FAQ
published: true
date: '2026-01-15T15:29:11.228Z'
tags: []
editor: visual
dateCreated: '2026-01-15T15:04:37.526Z'
---

# Frequently Asked Questions

## 1\. General Setup

**Q1. What is the recommended folder structure for a media server using TrueNAS Scale?**

Follow a logical setup like the [Trash Guides folder structure](https://trash-guides.info/), which includes separate folders for downloads, torrents, and media.

**Q2. Should I use Dockge, Portainer, or TrueNAS apps?**

Dockge is user-friendly for managing containers, while Portainer offers advanced networking features. Use TrueNAS apps if you prefer simplicity and native integration.

**Q3. How do I configure ACLs and permissions on TrueNAS?**

Set the dataset permissions to the `apps` user and group (UID/GID: 568) for compatibility with most Docker containers.

## 2\. VPN and Networking

**Q4. Do I need a VPN for qBittorrent or other torrent apps?**

Yes, especially for privacy. Avoid free VPNs and ensure your VPN provider supports port forwarding if using private trackers. I recommend [AirVPN](/AirVPN).

**Q5. How do I verify my VPN is working in qBittorrent?**

Use `curl ifconfig.me` in the container shell or add a magnet link to [ipleak.net's torrent test](https://ipleak.net/) and verify the IP.

**Q6. Can I set up my own VPN to connect remotely to services like Jellyfin or Nextcloud?**

Yes, use tools like [Tailscale](https://tailscale.com/) for an easy, secure VPN solution. Alternatively, try Nebula or Headscale for decentralized setups. You can also you [Wireguard](/wg-easy).

**Q7. What should I do if downloads stall in qBittorrent or trackers fail?**

Ensure your VPN is correctly configured. For private trackers, verify port forwarding settings. Use manual search in Radarr/Sonarr to find torrents with more seeders.

## 3\. Radarr/Sonarr Integration

**Q8. Why aren’t downloaded files moving from** `**/media/downloads**` **to** `**/media/movies**` **or** `**/media/tv**`**?**

Check the activity tab in Radarr/Sonarr for errors. If the file doesn’t match the movie/show, Radarr/Sonarr won’t move it. Adjust naming conventions or manual import. This also usually happens with permissions errors, so if your error says “Permission Denied” your permissions are incorrect.

**Q9. How do I configure Radarr/Sonarr to use hardlinks instead of copying files?**

Ensure downloads and final media directories are on the same dataset, and set Radarr/Sonarr to use hardlinks in the Media Management settings. See the [Folder Structure](/Folder-Structure) page for more details. 

**Q10. How do I add indexers and trackers to Radarr/Sonarr?**

Use [Prowlarr](/Prowlarr) for centralized indexer management. Make sure indexers support your content and region.

## 4\. qBittorrent Issues

**Q11. What does “host not found (non-authoritative)” mean in qBittorrent?**

It’s a DNS resolution issue. Ensure the container is properly connected to the VPN interface and DNS is configured in the VPN settings.

**Q12. How do I bind qBittorrent to the VPN interface?**

In the qBittorrent settings, bind the Web UI to the VPN interface (e.g., wg0). Refer to the guide for [advanced network settings](/qBittorrent).

**Q13. My downloads are very slow. What can I do?**

Check the number of seeders on the torrent. Use manual search to select torrents with more seeders.

## 5\. GPU and Hardware Acceleration

**Q14. How do I enable GPU passthrough for Jellyfin or other apps on TrueNAS?**

Enable NVIDIA GPU drivers in TrueNAS Apps settings, restart the system, and configure the app to use the GPU. If issues persist, reinstall TrueNAS and enable GPU drivers before adding apps.

**Q15. My GPU isn’t being used for transcoding in Jellyfin. What should I check?**

Ensure hardware transcoding is enabled in Jellyfin. Verify that the GPU is detected using `nvidia-smi`.

## 6\. Media Server Customizations

**Q16. Can I use Jellyseerr with Jellyfin?**

Yes, Jellyseerr supports Jellyfin integration. Ensure the URL and port are correctly set in the Jellyseerr settings.

**Q17. How do I allow friends to access my Jellyfin server?**

Use Tailscale or another VPN solution to give friends secure access.

## 7\. Common Errors and Solutions

**Q18. How do I fix “DNS resolution is currently unavailable” in Pi-hole?**

Check the Pi-hole DNS configuration and logs for errors. Restart the container if necessary.

**Q19. Why is my Docker container stuck in a restart loop?**

Check logs for specific errors. Common issues include incorrect environment variables, ports already in use, or missing volume paths.

**Q20. Why does Jellyfin or another app stop working outside my local network?**

Ensure your Tailscale or VPN setup is correctly configured. If using Tailscale, verify subnet routing or exit node settings.

## 8\. Advanced Topics

**Q21. How do I implement Trash Guides’ recommended setup on TrueNAS Scale?**

Set up datasets and shares for apps, downloads, and media. Configure Dockge or Docker Compose with the Trash Guides folder structure.

**Q22. How can I keep all my Docker containers updated?**

Use [Watchtower](/Watchtower) to automatically update Docker containers. Note that this does not work on apps from the TrueNAS Scale catalog.
