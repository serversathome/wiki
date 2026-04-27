---
title: Moonfin
description: A guide to dpeloying Moonfin
published: true
date: 2026-04-27T17:52:32.472Z
tags: 
editor: markdown
dateCreated: 2026-04-27T17:52:32.472Z
---

# What is Moonfin?

**Moonfin** is a collection of enhanced Jellyfin (and Emby) client applications for smart TVs and streaming devices. Built as forks of the official Jellyfin clients, Moonfin focuses on refined UI/UX, native Jellyseerr integration, cross-server content playback, and platform-specific optimizations — all while maintaining full compatibility with any standard Jellyfin server.

Available for Android TV / Fire TV, Roku, Samsung TV (Tizen), and LG TV (WebOS), Moonfin clients add features like a Featured Media Bar, theme music playback, pre-playback track selection, customizable navigation, MDBList rating sources, and the first **native** Jellyfin TV clients with built-in Jellyseerr/Seerr support.

# 1 · Server Requirements

Before installing any Moonfin client, make sure your media server is running a supported version:

| Server | Minimum Version | Status |
|--------|-----------------|--------|
| Jellyfin | 10.8.0+ | Full support |
| Emby | 4.8.0.0+ | Full support |


> 
> Moonfin clients are drop-in replacements for the official Jellyfin TV apps. If you already have a working Jellyfin/Emby server, no additional setup is needed on the server side to get started.
{.is-success}

# 2 · Install a Moonfin Client
# {.tabset}
## Android TV / Fire TV

The flagship Moonfin client supports Android TV 6.0+, Nvidia Shield, Fire TV / Fire TV Stick, and Chromecast with Google TV.

1. Download the latest APK from the [Releases page](https://github.com/Moonfin-Client/AndroidTV-FireTV/releases).
2. On your TV device, enable **Unknown Sources** or **Install Unknown Apps** in Settings.
3. Transfer the APK using one of:
   - A file manager app (e.g., **Send Files to TV**, **X-plore**)
   - A USB drive plugged into the device
   - **Downloader** app (Fire TV) using the GitHub release URL
4. Open the APK and follow the install prompts.
5. Launch Moonfin and connect to your Jellyfin / Emby server.

> 
> Moonfin for Android TV includes a built-in OTA update checker — once installed, future updates can be applied directly from inside the app.
{.is-info}

## Roku

The first and only Roku client with native Jellyseerr support. Works on Roku OS 9.1+ devices (2018 and newer), including Roku TV, Streaming Stick, Ultra, and Express.

1. On your Roku, navigate to **Settings → System → Advanced system settings → Developer options** and enable **Developer mode**. Note the IP address shown.
2. Set a developer username and password during the activation process.
3. Reboot the Roku when prompted.
4. Download the latest ZIP package from the [Roku Releases page](https://github.com/Moonfin-Client/Roku/releases).
5. On a computer on the same network, open `http://<roku-ip>` in a browser.
6. Log in with your developer credentials.
7. Click **Upload**, select the ZIP file, then click **Install**.
8. Launch Moonfin from the Roku home screen.

> 
> Roku developer mode requires factory-reset-level access. Only enable it on devices you control, and never enter your developer credentials on untrusted networks.
{.is-warning}

## Samsung TV (Tizen)

For Samsung Smart TVs running Tizen 4.0 or newer (2016+ models). Tizen 5.5+ is recommended for the best performance.

1. Download the latest WGT package from the [Tizen Releases page](https://github.com/Moonfin-Client/Tizen/releases).
2. The easiest install method is the [Samsung Jellyfin Installer](https://github.com/PatrickSt1991/Samsung-Jellyfin-Installer) — it handles certificate generation and TV pairing automatically.
3. Alternatively, install via **Tizen Studio**:
   - Install Tizen Studio with the TV Extensions package
   - Generate a Samsung certificate profile
   - Enable **Developer Mode** on your TV (varies by model — usually via the Apps panel)
   - Connect Tizen Studio to your TV's IP and use **Permit to install applications**
   - Use **Install** in the Device Manager to push the WGT
4. Launch Moonfin from the Apps screen on your TV.

> 
> Samsung's developer certificates expire after a fixed period (typically 1–2 years). If Moonfin disappears or refuses to launch after that time, you'll need to re-sign and reinstall the WGT.
{.is-warning}

## LG TV (WebOS)

For LG Smart TVs running WebOS 3.0 or newer (2016+ models). Includes optimizations for the LG Magic Remote.

1. Download the latest IPK package from the [WebOS Releases page](https://github.com/Moonfin-Client/WebOS/releases).
2. On your TV, sign in with an [LG Developer account](https://webostv.developer.lge.com/) and install the **Developer Mode** app from the LG Content Store.
3. Open the Developer Mode app, log in, and toggle **Dev Mode Status** on. Note the IP address and Passphrase.
4. On a computer, install [webOS Dev Manager](https://github.com/webosbrew/dev-manager-desktop) (community-maintained GUI).
5. Add your TV in Dev Manager using the IP and Passphrase, then use **Install** to push the IPK.
6. Launch Moonfin from the LG home screen.

> 
> Developer mode on WebOS expires every 50 hours by default. The TV will warn you to extend the session — just open the Dev Mode app and tap **Check for Updates** to reset the timer.
{.is-warning}

## tvOS (Coming Soon)

A Moonfin client for Apple TV is in development and not yet released. Watch the [main Moonfin organization](https://github.com/Moonfin-Client) for announcements.

# 3 · Initial Configuration

## 3.1 Connect to Your Server

On first launch, every Moonfin client walks through the same connection flow:

1. Choose **Add Server** on the welcome screen.
2. Enter your Jellyfin or Emby server URL (e.g., `http://192.168.1.100:8096` or `https://jellyfin.example.com`).
3. Moonfin auto-detects whether the server is Jellyfin or Emby — no manual toggle needed.
4. Sign in with your existing Jellyfin / Emby username and password.
5. Choose your default user profile and you're done.

> 
> Moonfin supports **Cross-Server Content Playback** — add multiple Jellyfin/Emby servers and Moonfin will present a unified library that lets you switch and play across them without re-authenticating.
{.is-success}

## 3.2 Optional: Jellyseerr / Seerr Integration

Moonfin's headline feature is native Jellyseerr support — browse trending content and request HD/4K movies and shows directly from your TV.

There are two ways to set it up:

**Option A — Moonfin Server Plugin (Recommended)**

1. Install the **Moonfin server plugin** on your Jellyfin server.
2. In the Jellyfin admin dashboard, configure your Jellyseerr/Seerr URL and API key in the plugin settings.
3. In your Moonfin client, go to **Settings → Plugin** and enable **Plugin Sync**.
4. Jellyseerr is now configured automatically — all requests are proxied through your Jellyfin server, so the client never needs direct access to Jellyseerr.


# 4 · Updates & Maintenance

- **Android TV / Fire TV** has built-in OTA updates — accept the prompt when a new version is detected.
- **Roku, Tizen, and WebOS** require manual reinstall via the same sideload process used for the initial install. Watch the relevant GitHub repo's Releases page for new versions, or subscribe to release notifications.
- All clients inherit upstream Jellyfin compatibility, so server-side updates do not require client updates unless a new Jellyfin major version is released.

