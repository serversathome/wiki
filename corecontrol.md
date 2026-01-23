---
title: CoreControl
description: A guide to deploying Core Control via docker compose
published: true
date: 2026-01-23T17:33:46.777Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:02.195Z
---

![corecontrol-light.png](/corecontrol-light.png)

# What is CoreControl?
![dashboard.png](/dashboard.png)
The only dashboard you'll ever need to manage your entire server infrastructure. Keep all your server data organized in one central place, easily add your self-hosted applications with quick access links, and monitor their availability in real-time with built-in uptime tracking. Designed for simplicity and control, it gives you a clear overview of your entire self-hosted setup at a glance.

[GitHub](https://github.com/crocofied/CoreControl)

# Installation
# {.tabset}
## Server Docker Compose
```yaml
services:
  web:
    image: haedlessdev/corecontrol:latest
    restart: unless-stopped
    ports:
      - 3005:3000
    environment:
      JWT_SECRET: c0568c603c61fc0614cd8cba5d880766aef0891baa1c73c459484495869199c87a4b56008e8278dfaf1945da101bf0b8935889cbd584ad1865ca434705b18632feb204428c06228de1001aaa97f3ae5981d750c6295a51c82cc75fd7475da9c23434f29c4de37e6dac3bcafa01d256fbde546071a37d3ebe70d0bcaeaa55b4b80b67552c88bea6dcfbc968091806ae147b8a91dbcabd151b9ee36c751476b4ad4dd4f611ed11cd3fe24fc92eb5e5978c725a93a8d814e6e85f3f578888d1305cdf9fd4ef0132cd393238d6497290ffa70e8f225ea47d0c04cb77e342705e77bfd0e1d79029ea818cca85a72ba269432ba128f8e892e83c8f32180a4a6e287112
      DATABASE_URL: postgresql://postgres:postgres@db:5432/postgres
      
  agent:
    image: haedlessdev/corecontrol-agent:latest
    restart: unless-stopped
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/postgres
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    volumes:
      - /mnt/tank/configs/corecontrol/:/var/lib/postgresql/data
    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U postgres
      interval: 2s
      timeout: 2s
      retries: 10
```

## Linux Endpoints Docker Compose
```yaml
services:
  glances:
    image: nicolargo/glances:latest
    container_name: glances
    restart: unless-stopped
    ports:
      - "61208:61208"
    pid: "host"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - GLANCES_OPT=-w --disable-webui
```

## Windows Endpoints Powershell
> 
> Be sure [Python](https://www.python.org/getit/) is installed an in your `$PATH`
{.is-info}

> Run as Administrator!
{.is-warning}


```powershell
# run this if you are prevented from running scripts: 
# Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force

# Run this as Administrator

# Define variables
$pythonPath = (Get-Command python).Source
$nssmUrl = "https://nssm.cc/release/nssm-2.24.zip"
$nssmZip = "$env:TEMP\nssm.zip"
$nssmExtractPath = "$env:ProgramFiles\nssm"
$glancesServiceName = "Glances"

# Step 1: Install Glances with web dependencies
Write-Host "Installing Glances with web support..."
pip install "glances[web]" -q

# Step 2: Download and extract NSSM
Write-Host "Downloading NSSM..."
Invoke-WebRequest -Uri $nssmUrl -OutFile $nssmZip

Write-Host "Extracting NSSM..."
Expand-Archive -Path $nssmZip -DestinationPath $nssmExtractPath -Force

# Find nssm.exe (assumes 64-bit Windows)
$nssmExe = Get-ChildItem "$nssmExtractPath\nssm-*\win64\nssm.exe" -ErrorAction Stop | Select-Object -First 1

# Step 3: Create the Glances service
Write-Host "Creating Glances service..."
& $nssmExe.FullName install $glancesServiceName $pythonPath "-m glances -w"

# Optionally set working directory (current user folder)
& $nssmExe.FullName set $glancesServiceName AppDirectory "$env:USERPROFILE"

# Step 4: Start the service
Write-Host "Starting Glances service..."
Start-Service $glancesServiceName

# Step 5: Add Windows Firewall inbound rule for Glances
Write-Host "Adding firewall rule for Glances on port 61208..."
New-NetFirewallRule -DisplayName "Allow Glances Web UI" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 61208 `
    -Action Allow `
    -Profile Any `
    -Description "Allows inbound TCP traffic for Glances web interface on port 61208"

Write-Host "Done. Glances is running on port 61208 and will auto-start on boot."
```

# Logging In
Corecontrol is available on `http://IP:3005`. The default user is `admin@example.com` and the default password is `admin`.

# Video

https://www.youtube.com/watch?v=wMSmOsZYmg0