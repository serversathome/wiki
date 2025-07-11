---
title: Netbird
description: A guide to installing and using Netbird
published: true
date: 2025-07-11T12:25:41.428Z
tags: 
editor: markdown
dateCreated: 2025-04-09T14:00:43.710Z
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

# 3 · Networking Quirks
When installing Netbird on a host running Docker containers as well as acting as a subnet router, other Netbird peers will not be able to reach the container IP:Port through the subnet.

For example, if I am running Netbird in a Docker container on my TrueNAS server (192.168.1.50 or 100.50.21.156 or DNS=truenas) routing for my 192.168.1.0/24 subnet, I will not be able to access my Jellfin server @ 192.168.1.50:8096 from other peers. I *would* be able to access it from the Netbird IP of 100.50.21.156:8096 or http://truenas:8096 due to the magic of Netbird, but many people would want to continue to use the IPv4 of 192.168.1.50:8096.

For that to work, install Netbird in an Incus LXC routing for 192.168.1.0/24 and you will be able to use 192.168.1.50:8096 to get to Jellyfin.

The other option is to edit the IP Tables of TrueNAS if you want to run Netbird in a docker container and have it reach all other docker containers. To do this, run these commands:

```bash
# Allow Docker containers to reach out through the VPN interface
sudo iptables -I FORWARD -i docker0 -o wt0 -j ACCEPT

# Allow return traffic from VPN back into Docker containers
sudo iptables -I FORWARD -i wt0 -o docker0 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Masquerade all traffic going out via the VPN (wt0)
sudo iptables -t nat -A POSTROUTING -o wt0 -j MASQUERADE
```

Also an interesting quirk: when running a Windows endpoint which also publishes a subnet, I am not able to RDP into the box using the private IPv4. For example, I expose the 10.240 route on a windows endpoint and when I try to RDP into 10.240.0.136 (the host) it does not work. It will work with the DNS name or the 100. address assigned by Netbird.

# <img src="/youtube.png" class="tab-icon"> 4 · Video
[](https://youtu.be/skbWnMSwZcE)
