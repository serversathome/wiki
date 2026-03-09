---
title: Kasm Workspaces
description: A guide to deploying Kasm Workspaces to Proxmox
published: true
date: 2026-03-09T16:22:20.792Z
tags: 
editor: markdown
dateCreated: 2026-02-25T10:19:47.919Z
---

# <img src="/kasm-workspaces.png" class="tab-icon"> What is Kasm Workspaces?

**Kasm Workspaces** is a container streaming platform that delivers browser-based access to desktops, applications, and web services. It lets you run isolated workspaces like full Linux desktops, web browsers, or development tools—all accessible through your web browser.

This guide covers setting up Kasm with **Proxmox autoscaling**, which automatically provisions and destroys Docker agent VMs based on user demand. When users launch workspaces, Kasm communicates with Proxmox to spin up new VMs. When sessions end, VMs are torn down after a configurable backoff period.

> 
> This guide requires both TrueNAS and Proxmox. The Kasm control plane runs on TrueNAS (via Dockge or the Apps catalog) while compute is offloaded to Proxmox VMs.
{.is-info}

#  Architecture Overview

| Component | Location | Description |
|-----------|----------|-------------|
| **Kasm Server** | Proxmox VM | Runs the Kasm Workspaces control plane |
| **Agent Template** | Proxmox | Ubuntu VM converted to template (Kasm clones this on demand) |
| **Autoscaled VMs** | Proxmox | 0-N clones spun up/down automatically based on user sessions |


# 1 · Deploy Kasm in Proxmox VM

## 1.1 Create VM in Proxmox

Create a new VM for the Kasm control plane:

| Setting | Value |
|---------|-------|
| OS | Ubuntu Server 24.04 |
| SCSI Controller | VirtIO SCSI Single |
| QEMU Agent | **Enabled** |
| Disk | 80GB minimum |
| CPU | 2+ cores |
| Memory | 4096MB minimum |
| Network Model | VirtIO (paravirtualized) |

> Get the Ubuntu ISO file [here](https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso)
{.is-info}

## 1.2 Install Ubuntu & Kasm

1. Install Ubuntu Server with OpenSSH enabled
2. After install, remove the ISO from the CD/DVD drive
3. SSH into the VM and install Kasm:

```bash
# Download Kasm
cd /tmp
curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.18.0.077388.tar.gz

# Extract
tar -xf kasm_release_1.18.0.077388.tar.gz

# Install (single server mode)
sudo bash kasm_release/install.sh
```

4. Follow the prompts and note the admin credentials displayed at the end

> 
> For the latest version, check [Kasm Downloads](https://www.kasmweb.com/downloads/)
{.is-info}

## 1.3 Configure Upstream Auth Address

After installation, log into the Kasm web UI:

1. Go to **Admin → Infrastructure → Deployment Zones**
2. Click **Edit** on your zone
3. Change **Upstream Auth Address** from `proxy` to your Kasm server's IP address (just the IP, no port)
4. Click **Save**

> 
> This step is critical. Autoscaled VMs need to know where to "phone home." If this remains set to `proxy`, agents cannot find Kasm and autoscaling silently fails.
{.is-danger}

# <img src="/proxmox.png" class="tab-icon"> 2 · Configure Proxmox

## 2.1 Create Resource Pool

1. Navigate to **Datacenter → Permissions → Pools → Create**
2. Name: `kasm-autoscale`

## 2.2 Create Dedicated User

1. Navigate to **Permissions → Users → Add**
2. Username: `kasm-autoscale`
3. Realm: **Proxmox VE authentication server**

> 
> The realm selection adds `@pve` automatically. Don't include it in the username field.
{.is-info}

## 2.3 Create API Token

1. Navigate to **Permissions → API Tokens → Add**
2. User: `kasm-autoscale@pve`
3. Token ID: `kasm` (or any name you choose)
4. Uncheck **Privilege Separation**
5. Click **Add**

> 
> Save the Token ID and Secret immediately—you cannot view them again. You'll need just the token name (e.g., `kasm`) for Kasm, not the full ID.
{.is-warning}

## 2.4 Create Role

Navigate to **Permissions → Roles → Create**:
- Name: `kasm-autoscale-role`
- Add these privileges:

  1. Datastore.AllocateSpace
  1. Pool.Audit
  1. SDN.Use
  1. VM.Allocate
  1. VM.Audit
  1. VM.Clone
  1. VM.Config.CDROM
  1. VM.Config.CPU
  1. VM.Config.Disk
   1. VM.Config.HWType
  1. VM.Config.Memory
  1. VM.Config.Network
  1. VM.Config.Options
  1. VM.GuestAgent.Unrestricted
  1. VM.PowerMgmt
  


## 2.5 Assign Permissions

Navigate to **Permissions → Add → User Permission** and create entries for each path:

| Path | User | Role |
|------|------|------|
| `/pool/kasm-autoscale` | kasm-autoscale@pve | kasm-autoscale-role |
| `/storage/local-lvm` | kasm-autoscale@pve | kasm-autoscale-role |
| `/sdn/zones/localnetwork` | kasm-autoscale@pve | kasm-autoscale-role |

> 
> Adjust `/storage/local-lvm` to match your storage name if different.
{.is-info}

> 
> The SDN zone path (`/sdn/zones/localnetwork`) is only needed if you're using Proxmox SDN. If you're using default bridge networking (vmbr0), you can skip this entry.
{.is-info}

# <img src="/ubuntu.png" class="tab-icon"> 3 · Create Agent Template

This VM lives in Proxmox and gets cloned by Kasm when it needs more compute capacity. It's completely separate from TrueNAS.

## 3.1 Create VM in Proxmox

| Setting | Value |
|---------|-------|
| OS | Ubuntu Server 24.04 |
| SCSI Controller | VirtIO SCSI Single |
| QEMU Agent | **Enabled** |
| Disk Bus | VirtIO Block |
| CPU Type | host |
| Network Model | VirtIO (paravirtualized) |
| Resource Pool | kasm-autoscale |


## 3.2 Install Ubuntu
- Uncheck **Set up this disk as an LVM group** (under storage configuration)
- Enable **OpenSSH server** during installation
- After install, remove ISO: **Hardware → CD/DVD → "Do not use any media"**

## 3.3 Prepare for Templating

SSH into the VM and run:

```bash
# Install and enable QEMU guest agent
sudo apt update
sudo apt install qemu-guest-agent -y
sudo systemctl start qemu-guest-agent
sudo systemctl enable qemu-guest-agent

# Reset machine IDs (required for templating)
sudo truncate -s 0 /etc/machine-id
sudo truncate -s 0 /var/lib/dbus/machine-id

# Shutdown
sudo shutdown now
```

## 3.4 Convert to Template

In Proxmox: **Right-click VM → Convert to Template**

> 
> Note the exact template name—you'll need it for Kasm configuration.
{.is-info}

# 4 · Configure Autoscaling in Kasm

All of the following is done in the **Kasm web UI**.

## 4.1 Create Pool

1. Go to **Admin → Infrastructure → Pools → Add**
2. Name: `Docker Agent Pool`
3. Type: **Docker Agent**
4. Click **Save**

## 4.2 Create AutoScale Config

Go to **Admin → Infrastructure → Pools → All AutoScale Configs → Add**

| Setting | Value |
|---------|-------|
| Name | Proxmox Docker Autoscale |
| AutoScale Type | Docker Agent |
| Pool | Docker Agent Pool |
| Downscale Backoff | 300 (or 100 for testing) |
| Standby Cores | 4 |
| Standby GPUs | 0 |
| Standby Memory | 6144 |
| Agent Cores Override | 4 |
| Agent Memory Override | 6144 |


Click **Next** to configure the VM Provider.

## 4.3 Configure VM Provider

| Setting | Value |
|---------|-------|
| Provider | Proxmox |
| Name | Proxmox Docker Config |
| Max Instances | 2 |
| Host | Your Proxmox IP (no `https://`) |
| Username | kasm-autoscale@pve |
| Token Name | Just the token name (e.g., `kasm`), not the full ID |
| Token Value | Your token secret (UUID format) |
| Verify SSL | Disabled |
| VMID Range | 1000 to 2000 |
| Full Clone | Disabled |
| Template Name | Exact name of your template (case-sensitive) |
| Cluster Node Name | *your cluster name* |
| Resource Pool Name | kasm-autoscale |
| Cores | 4 |
| Memory | 8 |
| Installed OS Type | Linux |
| Startup Script Path | /tmp |

> 
> The Token Name is just the name you gave the token (e.g., `kasm`), NOT the full token ID like `kasm-autoscale@pve!kasm`.
{.is-warning}


## 4.4 Startup Script

1. Get the latest startup script from Kasm's GitHub:

    ```
    https://github.com/kasmtech/workspaces-autoscale-startup-scripts/blob/release/1.18.1/docker_agents/ubuntu.sh
    ```

1. If your Kasm instance runs on a port other than 443 (e.g., 8443 or 30128), you must add the following lines to the end of the startup script:
    ```bash
    # Fix manager port for non-standard Kasm installations
    sed -i "/^manager:/,\$ s/public_port: 443/public_port: YOUR_PORT/" /opt/kasm/*/conf/app/agent/agent.app.config.yaml
    # Restart agent containers with corrected config
    cd /opt/kasm/current && docker compose -f docker/docker-compose.yaml up -d
    ```

	Replace `YOUR_PORT` with your actual Kasm port (e.g., 445 or 30128).

1. Click **Finish**.

# 5 · Testing

Once configured, Kasm will automatically provision VMs to meet the standby requirements.

1. Go to **Admin → Infrastructure → Docker Agents**
2. Watch for new agents appearing with status "Starting" → "Running"
3. Launch workspaces to trigger additional scaling
4. Close sessions and wait for downscale backoff to see VMs destroyed


# <img src="/youtube.png" class="tab-icon"> 6 · Video

*Video coming soon*