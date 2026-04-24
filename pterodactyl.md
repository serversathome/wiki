---
title: Pterodactyl & Wings
description: A guide to deploying Pterodactyl Panel and Wings
published: true
date: 2026-04-24T17:01:46.820Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:35.530Z
---

> UNDER CONSTRUCTION
{.is-danger}
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
    - Under **Devices**, click **Add → Filesystem** and fill in:

        | Field | Value |
        | --- | --- |
        | `Source` | `/mnt/<pool>/pterodactyl` (the host path) |
        | `Target` | `/var/lib/pterodactyl/volumes` (where Wings stores game-server data) |


1. Click your container → **Shell** and paste this to prep it:
    ```bash
    bash <<'EOF'
    until getent hosts deb.debian.org >/dev/null 2>&1; do sleep 1; done
    export DEBIAN_FRONTEND=noninteractive
    apt update
    apt install -y curl ca-certificates
    EOF
    ```
1. Then run the Pterodactyl installer as a **separate** paste (it's interactive and eats stdin if you chain anything after it):
    ```bash
    bash <(curl -s https://pterodactyl-installer.se)
    ```
1. In the menu, pick **Install both [0]**. Answer its prompts:
    - **Domain**: your domain, or just the container's LAN IP if you're LAN-only (check with `ip -4 -o addr show scope global`)
    - **Admin email / username / password**: whatever you want for the Panel root login
    - **Timezone**: yours
    - **SSL**: *no* if you're going LAN-only; *yes* if you gave it a real public-facing domain (uses Let's Encrypt, needs port 80 reachable from the internet)
    - **MariaDB / Redis**: let the installer set them up locally — defaults are fine
1. When it finishes it prints the URL for the Panel. Log in as the admin user you just created.

# 3 · Add Your Node in the Panel
This tells the Panel that Wings (on the same container) is the place to run game servers.

1. Panel → **Admin** (wrench icon) → **Nodes** → **Create New**
1. Fill in:

    | Field | Value |
    | --- | --- |
    | Name | anything, e.g. `local` |
    | Description | optional |
    | Location | create one first if the dropdown is empty |
    | FQDN | the container's hostname or IP (same as Panel URL usually) |
    | Communicate Over SSL | match whatever you picked during install |
    | Memory | how much RAM you'll let game servers eat (in MB) |
    | Disk | how much disk, in MB |
    | Daemon Port | `8080` (default) |
    | Daemon SFTP Port | `2022` (default) |
    | Daemon Server File Directory | `/var/lib/pterodactyl/volumes` |

1. **Create Node**
1. Click the node → **Configuration** tab. Copy the YAML it shows you.
1. Back in the container shell, paste it to `/etc/pterodactyl/config.yml`:
    ```bash
    nano /etc/pterodactyl/config.yml
    ```
    Paste, `Ctrl-O`, `Enter`, `Ctrl-X`.
1. Start and enable Wings:
    ```bash
    systemctl enable --now wings
    systemctl status wings --no-pager | head
    ```
    The status should show `active (running)` and the last log lines should say it connected to the panel. If you refresh the Nodes page in the Panel, the node now has a green heart.

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

