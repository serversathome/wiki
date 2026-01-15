---
title: Netbird
description: A guide to installing and using Netbird
published: true
date: 2026-01-15T15:30:19.388Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:37.607Z
---

# ![](/netbird.png){class="tab-icon"} What is Netbird?
NetBird combines a WireGuard®-based overlay network with Zero Trust Network Access, providing a unified open-source platform for reliable and secure connectivity

# 1 · Deploy Netbird
You must first go to [https://netbird.io](https://netbird.io) and click Get Started. After you create an account:
1. Navigate to **Setup Keys** ➡ **Create Setup Keys**
1. Give it a name
1. Switch the Reusable to ON
1. Set the expiry to `0`

# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  netbird:
    cap_add:
      - NET_ADMIN
    container_name: netbird
    environment:
      - NB_SETUP_KEY=
    image: netbirdio/netbird:latest
    network_mode: host
    restart: unless-stopped
    volumes:
      - ./netbird-client:/etc/netbird
```
Paste your Setup Key created from above in the `- NB_SETUP_KEY=` line.

## <img src="/linux.png" class="tab-icon"> Bare Metal Linux

```bash
curl -fsSL https://pkgs.netbird.io/install.sh | sh
```
Once this is installed, execute `netbird up --setup-key <key-value>` to start the connection using the setup key created above.

## <img src="/microsoft-windows.png" class="tab-icon"> Windows
1. Download the installer from [https://pkgs.netbird.io/windows/x64](https://pkgs.netbird.io/windows/x64)
1. Click on "Connect" from the NetBird icon in your system tray
a. Alternatively you can connect through the command line by executing `netbird up --setup-key <key-value>` using the setup key created above

## Mobile

Get the app from your respective app store:
- [Android *Google Play Store*](https://play.google.com/store/apps/details?id=io.netbird.client)
- [iOS *Apple App Store*](https://apps.apple.com/us/app/netbird-p2p-vpn/id6469329339)
{.links-list}

# 2 · Updating
On linux and mobile, updates are fairly easy through basic `apt upgrade -y` and app store downloads. However on Windows Netbird requires you to redownload the installer. As such, I have made this powershell script that can run as a Scheduled Task:

> Run this as admin!
{.is-warning}


```powershell
# IN the event running scripts is blocked run this command: Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force

# NetBird Silent Installer/Updater for All Users (with Auto-Start)


$repo = "netbirdio/netbird"
$assetNamePattern = "windows"
$downloadPath = "$env:TEMP\netbird_latest.exe"
$versionFile = "$env:ProgramData\netbird_version.txt"
$installPath = "C:\Program Files\NetBird\netbird.exe"

try {
    Write-Output "Checking latest NetBird version..."
    $releaseInfo = Invoke-RestMethod -Uri "https://api.github.com/repos/$repo/releases/latest" -Headers @{ "User-Agent" = "PowerShell" }
    $latestVersion = $releaseInfo.tag_name
    $asset = $releaseInfo.assets | Where-Object { $_.name -like "*$assetNamePattern*" -and $_.name -like "*.exe" } | Select-Object -First 1

    if (-not $asset) {
        Write-Output "Could not find a suitable release asset for Windows."
        pause
        return
    }

    $netbirdInstalled = Test-Path $installPath
    $currentVersion = if (Test-Path $versionFile) { Get-Content $versionFile -ErrorAction SilentlyContinue } else { "" }

    if (-not $netbirdInstalled) {
        Write-Output "NetBird is not installed. Installing latest version $latestVersion..."
    } elseif ($currentVersion -ne $latestVersion) {
        Write-Output "New version detected: $latestVersion (installed: $currentVersion). Updating..."
    } else {
        Write-Output "NetBird is up to date: $currentVersion"
        pause
        return
    }

    Write-Output "Downloading installer..."
    Invoke-WebRequest -Uri $asset.browser_download_url -OutFile $downloadPath

    # Kill running NetBird process if needed
    Get-Process -Name "netbird" -ErrorAction SilentlyContinue | Stop-Process -Force

    # Run installer silently (NSIS)
    Write-Output "Running silent install..."
    Start-Process -FilePath $downloadPath -ArgumentList "/S" -Wait

    # Save new version info
    Set-Content -Path $versionFile -Value $latestVersion

    # Install and start NetBird as a service
    if (Test-Path $installPath) {
        Write-Output "Starting NetBird service..."
        Start-Process -FilePath $installPath -ArgumentList "service install" -Wait
        Start-Process -FilePath $installPath -ArgumentList "service start" -Wait
    }

    Write-Output "Installation and startup complete. NetBird version: $latestVersion"
}
catch {
    Write-Output "An error occurred: $_"
}
```

# <img src="/youtube.png" class="tab-icon"> 4 · Video
[](https://youtu.be/skbWnMSwZcE)
