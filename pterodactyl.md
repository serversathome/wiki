---
title: Pterodactyl & Wings
description: A guide to deploying Pterodactyl Panel and Wings
published: true
date: 2026-04-24T17:39:37.062Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:35.530Z
---


# ![](/pterodactyl.png){class="tab-icon"} 1 · What Is Pterodactyl?
Pterodactyl is a free, open-source game server management panel. It gives you a web UI to deploy, start/stop, monitor, and manage game servers — Minecraft, Rust, ARK, Valheim, whatever — with per-server resource limits, a clean file manager, SFTP access, and user/subuser accounts so you can hand a server off to a friend without handing them root.

## 1.1 Why Use This?

Running a handful of game servers on a bare box works until you want more than one person managing them, or you want to cap how much CPU/RAM each one gets, or you want your kid to be able to stop and start the Minecraft server without you touching it. Pterodactyl solves all of that in one web UI.

Under the hood, **Wings** (the daemon) runs each game server inside its own Docker container with CPU/RAM/disk limits and isolated storage. The **Panel** is the web UI you actually interact with. On a homelab, running both on the same box (all-in-one) is the simplest setup and what this guide covers.

# <img src="/linuxcontainers.png" class="tab-icon"> 2 · TrueNAS 26 LXC Installation

1. **First time only**: go to **Virtualization → Containers → Global Configuration** and set:
    - **Preferred Pool** = the pool you want container rootfs on
    - **Bridge** = the bridge your LAN is on (e.g. `br1`). If you don't have one yet, make one in **Network → Interfaces** first.
1. Create a dataset for game server volumes: **Datasets → Add Dataset**, parent = your pool, name = `pterodactyl`, preset = **Generic**, save. No permission changes needed — the container runs privileged and Wings will manage ownership itself.
1. Create the container: **Virtualization → Containers → Add**
    - **Name**: `pterodactyl`
    - **Autostart**: on
    - **Image**: browse catalog → **debian / trixie / amd64 / default**
    - Click the **Advanced Options** button
    - **Idmap**: `Privileged`
    - **Capabilities**: `Allow All`
    - **Save**
    - Under **Filesystem Devices**, click **Add → Filesystem** and fill in:

        | Field | Value |
        | --- | --- |
        | `Source` | `/mnt/<pool>/pterodactyl` (the host path) |
        | `Target` | `/var/lib/pterodactyl/volumes` (where Wings stores game-server data) |
		
    - **Start** the container

1. Click your container → **Shell** and paste this to prep it:
    ```bash
    bash <<'EOF'
    until getent hosts deb.debian.org >/dev/null 2>&1; do sleep 1; done
    export DEBIAN_FRONTEND=noninteractive
    apt update
    apt install -y curl ca-certificates
    EOF
    ```
1. Then run the Pterodactyl installer:
    ```bash
      curl -o /tmp/ptero-install.sh https://pterodactyl-installer.se && bash /tmp/ptero-install.sh   
    ```
1. In the menu, pick **Install both [2]**. Answer its prompts:
		
    - **Database name**: `panel` (use default)
    - **Database username**: `pterodactyl` (use default)
    - **Password**: set a strong password
    - **Timezone**: yours
    - **Email (Let's Encrypt & Pterodactyl)**: enter any email
    - **Email (initial admin account)**: enter any email
    - **Username**: enter a username
    - **Frist name**: the first name of the admin user
    - **Last name**: last name of the admin user
    - **Password**: password for the admin user
    - **FQDN**: your domain, or just the container's LAN IP if you're LAN-only
    - **Configure UFW**: `no` (default)
    
		
1. After Panel is installed it will ask if you would like to proceed and install Wings. Say `yes` then answer:

     - **Are you sure you want to proceed?**: `yes` 
     - **Configure UFW**: `no` (default)
     - **Configure a user for database hosts?**: `yes`
     - **Configure MySQL to be accessed externally?**: `yes`
     - **Enter the panel address**: leave blank (default)
     - **Database host username**: `pterodactyluser` (default)
     - **Database host password**: enter a strong password
     - **Configure HTTPS**: `no` (default)
     
1. When it finishes it prints the URL for the Panel. Log in as the admin user you just created.

# 3 · Add Your Node in the Panel
This tells the Panel that Wings (on the same container) is the place to run game servers.

1. Navigate to **Locations** in the left sidebar and create a location with and short code
1. Panel → **Admin** (gear icon) → **Nodes** → **Create New**

1. Fill in:

    | Field | Value |
    | --- | --- |
    | Name | anything, e.g. `local` |
    | Description | optional |
    | Location | the one you just made |
    | FQDN | the container's hostname or IP (same as Panel URL usually) |
    | Communicate Over SSL | match whatever you picked during install |
    | Daemon Server File Directory | `/var/lib/pterodactyl/volumes` (default)|
    | Memory | how much RAM you'll let game servers eat (in MB) |
    | Disk | how much disk, in MB |
    | Daemon Port | `8080` (default) |
    | Daemon SFTP Port | `2022` (default) |


1. **Create Node**
1. Click the node → **Configuration** tab
1. Click the **Generate Token** button in the **Auto-Deploy** box
1. Back in the container shell, paste the command
1. Next paste this to start and enable Wings:
    ```bash
		systemctl enable --now wings && systemctl status wings --no-pager | head
    ```

If you refresh the Nodes page in the Panel, the node should now have a green heart.

# 4 · Add Allocations (IP:Port pairs for game servers)
Each game server needs at least one IP:port allocation.

1. Node → **Allocation** tab → **Assign New Allocations**
1. **IP Address**: the container's LAN IP
1. **Ports**: a range for game servers (e.g. `25565-25575` for Minecraft, or a wider range). You can add more later.
1. **Submit**

# 5 · Create Your First Server
1. Panel → **Servers** → **Create New**
1. Pick an owner, a Node, an Allocation, and a game Egg (Minecraft Java Vanilla is a safe first test — Pterodactyl ships with it by default).
1. Give it RAM/CPU/disk limits, hit **Create**.
1. Watch Wings pull the Docker image, install the server files, and start it up.

# 6 · Router / Firewall
Because the container is on your LAN bridge (not NATed), its IP is reachable from your LAN directly. To expose game servers to the internet, forward the allocation ports on your router to the container's IP (no port-forward magic needed on TrueNAS itself). 

