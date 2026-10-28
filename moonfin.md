---
title: Moonfin
description: A guide to dpeloying Moonfin
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-04-27T17:52:32.472Z
---

# What is Moonfin?

**Moonfin** is a collection of enhanced Jellyfin (and Emby) client applications for smart TVs and streaming devices. Built as forks of the official Jellyfin clients, Moonfin focuses on refined UI/UX, native Jellyseerr integration, cross-server content playback, and platform-specific optimizations — all while maintaining full compatibility with any standard Jellyfin server.

Available for Android (phone and TV), Fire TV, Roku, Samsung TV (Tizen), LG TV (WebOS), Xbox, iOS, tvOS, macOS, Windows, Linux and the web, Moonfin clients add features like a Featured Media Bar, theme music playback, pre-playback track selection, customizable navigation, MDBList rating sources, and the first **native** Jellyfin TV clients with built-in Jellyseerr/Seerr support.

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

> The clients were consolidated in 2026. Android, Android TV, Fire TV (Fire OS), iOS, tvOS, macOS, Windows and Linux now ship from [Moonfin-Core](https://github.com/Moonfin-Client/Moonfin-Core/releases). Samsung (Tizen), LG (webOS), Fire TV (Vega OS) and Xbox ship from [Smart-TV](https://github.com/Moonfin-Client/Smart-TV/releases). The old `AndroidTV-FireTV`, `Tizen` and `WebOS` repositories are gone.
{.is-info}

# {.tabset}
## Android TV / Fire TV

The Android client supports Android 6.0+ phones and TVs, Nvidia Shield, Fire TV / Fire TV Stick, and Chromecast with Google TV.

**Store install (recommended)**

- **Android / Android TV**: install Moonfin from [Google Play](https://play.google.com/store/apps/details?id=org.moonfin.androidtv).
- **Fire TV**: install Moonfin from the [Amazon Appstore](https://www.amazon.com/Moonfin/dp/B0H76BG3DW). This listing also covers newer Vega OS Fire TV devices.

**Sideload (optional)**

1. Download the latest APK from the [Moonfin-Core Releases page](https://github.com/Moonfin-Client/Moonfin-Core/releases).
2. On your TV device, enable **Unknown Sources** or **Install Unknown Apps** in Settings.
3. Transfer the APK using one of:
   - A file manager app (e.g., **Send Files to TV**, **X-plore**)
   - A USB drive plugged into the device
   - **Downloader** app (Fire TV) using the GitHub release URL
4. Open the APK and follow the install prompts.
5. Launch Moonfin and connect to your Jellyfin / Emby server.

## Roku

A Roku client with native Seerr support. Works on Roku OS 9.1+ devices (2018 and newer), including Roku TV, Streaming Stick, Ultra, and Express.

**Store install (recommended)**

Install Moonfin from the [Roku Channel Store](https://channelstore.roku.com/details/92a83c9f4112b76a7bcee3dc076254ca:3251a91bf7af7339652d5409ccfdcb39/moonfin).

**Sideload (optional)**

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

1. Download the latest WGT package from the [Smart-TV Releases page](https://github.com/Moonfin-Client/Smart-TV/releases).
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

**Homebrew Channel install**

If your TV has the webOS Homebrew Channel, install Moonfin from the [webOS Homebrew repo](https://repo.webosbrew.org/apps/org.moonfin.webos/).

**Developer Mode sideload**

1. Download the latest IPK package from the [Smart-TV Releases page](https://github.com/Moonfin-Client/Smart-TV/releases).
2. On your TV, sign in with an [LG Developer account](https://webostv.developer.lge.com/) and install the **Developer Mode** app from the LG Content Store.
3. Open the Developer Mode app, log in, and toggle **Dev Mode Status** on. Note the IP address and Passphrase.
4. On a computer, install [webOS Dev Manager](https://github.com/webosbrew/dev-manager-desktop) (community-maintained GUI).
5. Add your TV in Dev Manager using the IP and Passphrase, then use **Install** to push the IPK.
6. Launch Moonfin from the LG home screen.

> 
> Developer mode on WebOS expires every 50 hours by default. The TV will warn you to extend the session — just open the Dev Mode app and tap **Check for Updates** to reset the timer.
{.is-warning}

## Apple (iOS / tvOS / macOS)

Moonfin is available for iPhone, iPad, Apple TV and Mac from the [App Store](https://apps.apple.com/app/moonfin/id6761283970). On macOS you can also install it with Homebrew:

```bash
brew install --cask moonfin
```

## Desktop / Xbox / Web

- **Windows**: install from the [Microsoft Store](https://apps.microsoft.com/detail/9plb327dd51d), or download the installer from [Moonfin-Core releases](https://github.com/Moonfin-Client/Moonfin-Core/releases).
- **Linux**: binaries are in [Moonfin-Core releases](https://github.com/Moonfin-Client/Moonfin-Core/releases), and Arch users can install [`moonfin-bin`](https://aur.archlinux.org/packages/moonfin-bin) from the AUR.
- **Xbox**: install from the [Microsoft Store](https://apps.microsoft.com/detail/9plb327dd51d), or sideload the ZIP from [Smart-TV releases](https://github.com/Moonfin-Client/Smart-TV/releases) in Developer Mode ([guide](https://github.com/Moonfin-Client/Smart-TV/wiki/Installation-and-Sideloading#xbox)).
- **Web**: install the Moonbase server plugin (section 3.2). It serves the Moonfin web app as an installable PWA at `http://<jellyfin-server>/Moonfin/Web/`.

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

Seerr support requires the **Moonbase** server plugin, which upstream also strongly recommends for all users (it handles settings sync, the Seerr proxy, MDBList/TMDB and the hosted web app).

1. Install the [Moonbase plugin](https://github.com/Moonfin-Client/Plugin) on your Jellyfin server.
2. In the Jellyfin admin dashboard, go to **Dashboard → Plugins → Moonfin** and configure your Jellyseerr/Seerr URL and API key.
3. In your Moonfin client, go to **Settings → Plugin** and enable **Plugin Sync**.
4. Jellyseerr is now configured automatically — all requests are proxied through your Jellyfin server, so the client never needs direct access to Jellyseerr.


# 4 · Updates & Maintenance

- **Store installs** (Google Play, Amazon, Roku Channel Store, App Store, Microsoft Store) update through the store.
- **Sideloaded installs** (APK, Tizen WGT, webOS IPK, Roku ZIP) require a manual reinstall via the same sideload process used for the initial install. Watch the relevant GitHub repo's Releases page for new versions, or subscribe to release notifications.
- All clients inherit upstream Jellyfin compatibility, so server-side updates do not require client updates unless a new Jellyfin major version is released.

