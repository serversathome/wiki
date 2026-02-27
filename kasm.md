---
title: Kasm Workspaces
description: A guide to deploying Kasm Workspaces to Proxmox
published: true
date: 2026-02-27T16:06:27.439Z
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

# Architecture Overview

| Component | Location | Description |
|-----------|----------|-------------|
| **Kasm Server** | TrueNAS | Runs Kasm Workspaces (Dockge or TrueNAS App) |
| **Agent Template** | Proxmox | Ubuntu VM converted to template (Kasm clones this on demand) |
| **Autoscaled VMs** | Proxmox | 0-N clones spun up/down automatically based on user sessions |


# 1 · Deploy Kasm on TrueNAS

Choose your preferred deployment method:

# {.tabset}

## Dockge (LinuxServer.io)

### 1.1 Create Datasets

Before deploying Kasm, create datasets for persistent storage:

1. Click **Datasets** on the left
2. Select your configs dataset (e.g., `tank/configs`)
3. Click **Add Dataset**, name it `kasm`, click **Save**
4. Select the new `kasm` dataset and create two child datasets:
   - `opt` — Kasm application data, database, certificates
   - `profiles` — User profile persistence

#### Permissions

For each dataset (`kasm/opt` and `kasm/profiles`):

1. Click the dataset, find **Permissions** on the lower left, click **Edit**
2. Leave User as `root`
3. Change Group to `apps`
4. Enable all group permission checkboxes
5. Disable all permission checkboxes for Other
6. Click **Apply Group**, then **Save**

### 1.2 Deploy with Dockge

> 
> If you've never used Dockge before, check out the Dockge setup guide on the wiki.
{.is-info}

1. In Dockge, click the **+** button to create a new stack
2. Name it `kasm`
3. Paste the following compose file:

```yaml
services:
  kasm:
    image: lscr.io/linuxserver/kasm:latest
    container_name: kasm
    privileged: true
    environment:
      - KASM_PORT=443
      - DOCKER_MTU=1500
      - DOCKER_SUBNET=172.17.0.0/24
    volumes:
      - /mnt/tank/configs/kasm/opt:/opt
      - /mnt/tank/configs/kasm/profiles:/profiles
      - /dev/input:/dev/input
      - /run/udev/data:/run/udev/data
    ports:
      - 3000:3000
      - 445:443
    restart: unless-stopped
```

> 
> Port 443 is used by the TrueNAS web UI, so we map Kasm to port 445 instead.
{.is-info}

> 
> If your pool is named something besides `tank`, change the left side of the volume paths.
{.is-info}

4. Click **Deploy**

#### First Run Setup

1. Access the install wizard at `https://<your-ip>:3000`
2. Accept the EULA
3. Set your admin password
4. Wait for installation to complete

After setup, access the Kasm web UI at `https://<your-ip>:445`

> 
> Port 3000 is only used for initial setup. After installation, you'll use port 445.
{.is-info}

### 1.3 Configure Upstream Auth Address

After installation, log into the Kasm web UI:

1. Go to **Admin → Infrastructure → Zones**
2. Click **Edit** on your zone
3. Change **Upstream Auth Address** from `proxy` to your server's IP address (just the IP, no port)
4. Click **Save**

> 
> This step is critical. Autoscaled VMs need to know where to "phone home." If this remains set to `proxy`, agents cannot find Kasm and autoscaling silently fails.
{.is-danger}

## TrueNAS Apps

### 1.1 Create Datasets

Before deploying Kasm, create datasets for persistent storage:

1. Click **Datasets** on the left
2. Select your configs dataset (e.g., `tank/configs`)
3. Click **Add Dataset**, name it `kasm`, click **Save**
4. Select the new `kasm` dataset and create two child datasets:
   - `opt` — Kasm application data, database, certificates
   - `profiles` — User profile persistence

#### Permissions

For each dataset (`kasm/opt` and `kasm/profiles`):

1. Click the dataset, find **Permissions** on the lower left, click **Edit**
2. Leave User as `root`
3. Change Group to `apps`
4. Enable all group permission checkboxes
5. Disable all permission checkboxes for Other
6. Click **Apply Group**, then **Save**

### 1.2 Install the App

1. Navigate to **Apps** in the TrueNAS UI
2. Search for "Kasm Workspaces"
3. Click **Install**

#### Storage Configuration

Change both storage types from ixVolume to **Host Path**:

| Setting | Host Path |
|---------|-----------|
| Kasm Workspaces Opt Storage | `/mnt/tank/configs/kasm/opt` |
| Kasm Workspaces Profiles Storage | `/mnt/tank/configs/kasm/profiles` |

> 
> If your pool is named something besides `tank`, adjust the paths accordingly.
{.is-info}

#### Network Configuration

Leave the default ports:
- **WebUI Port**: 30128
- **Setup Port**: 30129

Click **Save** and wait for the app to deploy.

Once deployed, access the Kasm web UI at `https://<your-ip>:30128`. Default credentials are shown in the app logs.

### 1.3 Configure Upstream Auth Address

After deployment, log into the Kasm web UI:

1. Go to **Admin → Infrastructure → Zones**
2. Click **Edit** on your zone
3. Change **Upstream Auth Address** from `proxy` to your server's IP address (just the IP, no port)
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
2. Username: `kasm-autoscale@pve`
3. Realm: **Proxmox VE authentication server**

## 2.3 Create API Token

1. Navigate to **Permissions → API Tokens → Add**
2. User: `kasm-autoscale@pve`
3. Uncheck **Privilege Separation**
4. Click **Add**

> 
> Save the Token ID and Secret immediately—you cannot view them again.
{.is-warning}

## 2.4 Create Role

Navigate to **Permissions → Roles → Create**:
- Name: `kasm-autoscale-role`
- Add these privileges:


|---|---|---|
| Pool.Audit | VM.Allocate | Datastore.AllocateSpace |
| SDN.Use | VM.Audit | VM.Clone |
| VM.Config.CDROM | VM.Config.CPU | VM.Config.Disk |
| VM.Config.HWType | VM.Config.Memory | VM.Config.Network |
| VM.Config.Options | VM.Monitor | VM.PowerMgmt |


## 2.5 Assign Permissions

Navigate to **Permissions → Add → User Permission** and create three entries:

| Path | User | Role |
|------|------|------|
| `/sdn/zones/localnetwork` | kasm-autoscale@pve | kasm-autoscale-role |
| `/storage/local-lvm` | kasm-autoscale@pve | kasm-autoscale-role |
| `/pool/kasm-autoscale` | kasm-autoscale@pve | kasm-autoscale-role |


> 
> Adjust `/storage/local-lvm` to match your storage name if different.
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

> Get the Ubuntu ISO file [here](https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso)
{.is-info}

## 3.2 Install Ubuntu

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
| Token Name | (your token name) |
| Token Value | (your token secret) |
| Verify SSL | Disabled |
| VMID Range | 1000 to 2000 |
| Full Clone | Disabled |
| Template Name | (exact name of your template) |
| Cluster Node Name | pve |
| Resource Pool Name | kasm-autoscale |
| Cores | 4 |
| Memory | 8192 |
| Installed OS Type | Linux |
| Startup Script Path | /tmp |


## 4.4 Startup Script

1. Get the latest startup script from Kasm's GitHub:

    ```
    https://github.com/kasmtech/workspaces-autoscale-startup-scripts
    ```

1. Use the Linux/Docker agent script. Paste it into the **Startup Script** field.


    > As of Kasm v1.17+, remove the NGINX cert/key lines from the script—they're no longer needed.
   <!-- {blockquote:.is-info} -->

1. Click **Finish**.

# 5 · Testing

Once configured, Kasm will automatically provision VMs to meet the standby requirements.

1. Go to **Admin → Infrastructure → Docker Agents**
2. Watch for new agents appearing with status "Starting" → "Running"
3. Launch workspaces to trigger additional scaling
4. Close sessions and wait for downscale backoff to see VMs destroyed


# <img src="/youtube.png" class="tab-icon"> 6 · Video

*Video coming soon*