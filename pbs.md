---
title: Proxmox Backup Server
description: A guide to deploying Proxmox Backup Server
published: true
date: 2026-01-15T15:30:38.291Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:02.183Z
---

# ![](/proxmox.png){class="tab-icon"} 1 · What Is Proxmox Backup Server?
Proxmox Backup Server (PBS) is an enterprise backup solution, for backing up and restoring VMs, containers, and physical hosts. By supporting incremental, fully deduplicated backups, Proxmox Backup Server significantly reduces network load and saves valuable storage space. With strong encryption and methods of ensuring data integrity, you can feel safe when backing up data, even to targets which are not fully trusted.

## 1.1 Why Use This?

Using **Proxmox Backup Server (PBS)** is generally better than backing up Proxmox VMs directly to a TrueNAS NFS share because PBS provides deduplication and incremental backups. Instead of saving full VM backup files every time, PBS stores data in chunks and only uploads the blocks that changed. This dramatically reduces storage usage and speeds up backup jobs, especially when backing up multiple similar VMs or running daily schedules.

PBS also adds features that NFS-backed backups simply don’t have: built-in encryption, automatic verification to detect corruption, and flexible retention policies (daily/weekly/monthly). These features make backups safer, more space-efficient, and easier to maintain without manual cleanup or scripting. NFS backups, by comparison, are full image files with no deduplication, no integrity checks, and no retention logic.

Finally, PBS integrates directly into the Proxmox interface and supports fast, granular restores including disk-level and file-level recovery. NFS backups can only restore full VMs and offer no dedup stats, chunk verification, or incremental versioning. In short: PBS gives you faster backups, safer data, and massive storage savings, all while still letting you store the data on TrueNAS if you want.


# 2 · Installation
# {.tabset}
## VM
1. Go to the [official Proxmox Download Page](https://www.proxmox.com/en/downloads/proxmox-backup-server/iso) to grab the latest ISO of Proxmox Backup Server
1. Install the ISO. PBS uses very little resources - on my machine I give it 1 CPU threads, 2 GB of memory and a 10 GB hard drive. 
1. Boot into the webUI using `https://{IP}:8007`, **root** as the user name and the password you selected during install

## <img src="/linuxcontainers.png" class="tab-icon"> Fangtooth LXC

1. Create a new Instance from the **Debian trixie (amd64, default)** image with all default settings
1. Once its running, shell into the container and run these commands:
    ```bash
    echo "deb http://download.proxmox.com/debian/pbs trixie pbs-no-subscription" | sudo tee -a /etc/apt/sources.list
    sudo apt install wget -y
    wget https://enterprise.proxmox.com/debian/proxmox-release-trixie.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-trixie.gpg
    sudo apt update && apt upgrade -y
    sudo apt install -y whiptail apt-utils coreutils bash proxmox-widget-toolkit nano nfs-common cron
    sudo apt update && apt install proxmox-backup-server -y
    echo "https://$(ip -4 addr show $(ip route | grep default | awk '{print $5}') | grep inet | awk '{print $2}' | cut -d/ -f1):8007"
    passwd root
    ```

1. Once the commands have completed, **enter a root password**
1. The commands will print the IP address just before asking you to create a root password
1. Go to the printed IP address to access the webui
1. Login to the webui as **root** and the password you just created

# 3 · Post Install Script
It is recommended you run the [Proxmox Backup Server Post Install Script](https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pbs-install) from [helper scripts](https://community-scripts.github.io/ProxmoxVE/) in the PBS shell using all default options:
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pbs-install.sh)"
```
# 4 · Adding a TrueNAS Dataset to PBS
1. Create a dataset on TrueNAS with generic permissions
1. Change the permissions to user:group = `backup:backup 770` as shown below:
![screenshot_from_2025-06-12_23-28-32.png](/screenshot_from_2025-06-12_23-28-32.png)

# {.tabset}
## VM
3. Create an NFS share from the dataset
a. Use the **Advanced Options** to set the **Maproot User** to `root`
1. Run this command inside the PBS shell changing the IP, path, and mount point for your server:
    ```bash
    echo "10.99.0.191:/mnt/tank/pbs /backup nfs vers=3,nouser,atime,auto,retrans=2,rw,dev,exec 0 0" >> /etc/fstab
    ```
1. Reboot PBS

## LXC
3. Mount the dataset inside the LXC by running the following command in the TrueNAS Shell:
```bash
sudo incus exec <container-name> -- mkdir -p /backup && sudo incus config device add <container-name> mydataset disk source=/mnt/tank/pbs path=/backup shift=true

```
|Field | Value|
| ---| ----|
| `container-name` | name of the LXC |
| `mkdir` | the name of the mount point to create inside the container|
| `source=` | path on the TrueNAS host|
| `path=` | the mount point from the mkdir above |

# 5 · Add the Datastore to PBS
1. Navigate in the PBS panel on the left and click **Add Datastore** 
1. Give the datastore a name and use the path you mounted from the previous steps as the **Backing Path**
1. Click **Add**

# 6 · Adjust User Permissions
1. Navigate to **Configuration** → **Access Control** in the left pane
1. Add a new user leaving all options as default
1. Click **Add**
1. Click the name of the Datastore you just added
1. Click **Permissions** in the top bar
1. Click **Add**
1. Select the new user and **Datastore Admin** as **Role**

# 7 · Add your PBS Server to Proxmox

1. Navigate to your Proxmox VE machine 
1. Click **Datacenter** then **Storage** in the left pane
1. Click **Add** and select **Proxmox Backup Server**
1. Use the values in the below table to fll in the fields:

| Field | Value |
| --- | --- |
| ID | Any name you choose | 
| Server | the IP of the PBS server (no port number needed) | 
| Username | the user you created (example: user@pbs) |
| Password | the password you chose | 
| Datastore| the name of the Datastore you picked on PBS |
| Fingerprint | to get this, go to PBS Dashboard and at the top look for the blue button that says **Show Fingerprint** |

# <img src="/cloudflare.png" class="tab-icon"> 8 · Connecting With Cloudflare Tunnels
A note for those using a CF tunnel to connect to your WebUI, make these changes to **Additional Application Settings** while editing your tunnel to ensure proper connection:
1. Turn the **No TLS Verify** to `ON`
1. Turn the **Disable Chucked Encoding** to `ON`

> If you have **Botfight Mode** enabled you must create a rule to bypass for your IP
{.is-info}

# <img src="/youtube.png" class="tab-icon"> Video Walkthrough
[](https://youtu.be/lUWB-Dash9M)