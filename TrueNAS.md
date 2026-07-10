---
title: TrueNAS Community Edition
description: This article will describe how to set up a TrueNAS server to be compatible will services described in this wiki.
published: true
date: 2026-07-10T15:41:15.396Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:56.437Z
---

> This page was built to describe TrueNAS CE Goldeye 25.10.4
{.is-info}

# <img src="/truenas.png" class="tab-icon"> Installation
https://youtu.be/cA8fZ-lfgaA?feature=shared


# <img src="/youtube.png" class="tab-icon"> YouTube Basic Walkthrough

See the playlist here: `https://youtube.com/playlist?list=PL6zQmF2gDqDT7SHyBe7ni1P2S4NzyJpD6`

# {.tabset}

## Dashboard
### Dashboard
https://youtu.be/wbeAWq8WiqE?feature=shared

## Storage
### Storage
https://youtu.be/YgTFRrwJnwY?feature=shared

### Renaming a Pool

> This is strongly recommended if you have a pool name with spaces or capitalization in it
{.is-warning}

> If you have any apps or shares accessing the pool, stop them before starting these steps
{.is-danger}


1. Navigate to **Storage → Pool**
1. Click on the **Export/Disconnect** button to export the pool *without destroying any of the data*:

	![](/screenshot_from_2024-12-16_10-18-04.png)

1. Next, in the shell, run these commands **as root** replacing the original name of the pool with the new name you have chosen.:

    ```bash
    zpool import original_name new_name
    ``` 
1. to confirm the new name is working:
    ```bash
    zpool status new_name
    ``` 
1. so we can import it in the GUI again:
    ```bash
    zpool export new_name
    ```
1. Lastly, navigate to the **Storage → Pool** tab and click the button for **Import Pool** in the top right corner. Select the new pool you have just renamed.
> 
> If you had any apps using hostpath for the old pool they will have to be edited/recreated. Same goes for rsync tasks, shares, snapshots, etc.  
{.is-warning}

https://youtu.be/mm3lyPQcseE

### Adding a Single Disk

To [add a single disk](https://www.truenas.com/docs/scale/scaletutorials/storage/managepoolsscale/#extending-a-raidz-vdev) to an existing RAIDZ(1,2,3) pool, use the **Extend** button in the devices menu.

![devicesmirrorvdevselected.png](/devicesmirrorvdevselected.png)

Note that you will **not** be able to gain 100% of the usable space from that disk until you run the [rebalancing script](https://github.com/markusressel/zfs-inplace-rebalancing). You can calculate how much capacity you will gain by using the [calculator](https://www.truenas.com/docs/references/extensioncalculator/).

To view progress of the expansion, run this command in the shell: 
```bash
zpool status pool_name
```
Watch Lawrence do it:
https://youtu.be/uPCrDmjWV_I


## Datasets
### Datasets
https://youtu.be/27MhvLKBKtQ?feature=shared

For snapshots and rollbacks at a granular level, datasets should be set up in the pool for each individual app.

![](/screenshot_from_2025-01-20_10-50-23.png)

### Host Path Config Storage

Apps running on Scale should use the Host Path Config option to store their data. You will find this option in the right hand menu under **Storage Configuration** > *Type*. This allows for the rapid redeployment of apps with no loss to their configurations, which can be considerable for some. Each app uses its respective host path from the example above. For a video explanation and example of this:  
https://youtu.be/JZ9zbcyLcDo

### Permissions

In order for the apps to write to the dataset properly, make sure to select the ‘Apps’ option for the dataset preset:

![](/screenshot_from_2024-10-21_16-10-36.png)

Once the dataset has been created, you can still modify its permissions like below:

![](/screenshot_from_2024-02-23_11-52-29.png)

For a video walkthrough on permissions:  
https://youtu.be/qAGN0_73cV4

## Shares
### Shares
https://youtu.be/RAlJ3fcktMQ

## Data Protection

**Drive Health Management** is how TrueNAS keeps an eye on your physical disks. Since TrueNAS **25.10 (Goldeye)**, the SMART test scheduling and results screens were removed from the GUI. SMART itself was not removed. TrueNAS still polls SMART data on every drive and still raises alerts on critical health indicators. What you lost is the recurring **self-tests** (the scheduled short and long scans), and this page shows you how to set those back up, how to read the results, and the other tasks that extend the life of your drives. This matters more than ever with hard drive prices up roughly 50% due to the AI storage crunch.
 
> 
> Setting up your own SMART self-tests is optional but recommended. A long test is the only thing that reads every sector, including empty ones, catching bad spots before your data lands on them.
{.is-info}
 
### 1 · What changed in 25.10
 
TrueNAS 25.10 removed the SMART UI (NAS-134927) and the built-in test scheduler (NAS-135020). The `smartmontools` binaries are still installed, so scripts and third-party tools keep working. Existing scheduled tests from 25.04 and earlier were **automatically migrated to cron jobs** during the upgrade, so check your Cron Jobs list before creating new ones.
 
> 
> iX has announced they are working on bringing back API endpoints and UI trigger buttons for SMART testing in a future release. Until that ships, use one of the methods below.
{.is-info}
 
### 2 · Set up automatic SMART tests

#### midclt (recommended)
 
TrueNAS ships a middleware command that already knows about every disk in the system, so it is the cleanest way to test everything at once. Open the **Shell** and run:
 
```bash
midclt call disk.smart_test SHORT '["*"]'
```
 
Swap `SHORT` for `LONG` to run the full surface scan:
 
```bash
midclt call disk.smart_test LONG '["*"]'
```
 
The `["*"]` is a wildcard meaning "all disks."
 
> 
> A successful call returns `null`. That is not an error, it just means the tests kicked off. This is also why you may get "produced the following output: null" emails after upgrading.
{.is-warning}
 
To schedule it, go to **System Settings → Advanced → Cron Jobs → Add**, paste the command, set the user to `root`, and set a schedule. A good baseline is **SHORT daily** and **LONG weekly or monthly**.
 
#### Cron job (smartctl)
 
If you would rather stay in pure `smartctl` and not touch the TrueNAS API, this portable loop tests every drive the scan finds, spinning disk, SATA SSD, or NVMe alike:
 
```bash
for dev in $(smartctl --scan | cut -d' ' -f1); do smartctl -t short "$dev"; done
```
 
Add it under **System Settings → Advanced → Cron Jobs**, run as `root`, on a daily schedule. Make a second job with `-t long` for the weekly or monthly full scan.
 
#### Scrutiny (GUI)
 
For a full dashboard with per-drive history, scheduled tests, and alerting, install **Scrutiny** from the TrueNAS apps catalog:
 
1. Navigate to **Apps** in the TrueNAS UI
2. Search for **Scrutiny**
3. Click **Install**, then open the **Web UI**
> 
> Scrutiny was forked and is under active development again, so it is a safe pick today. Once it is collecting data you get trend lines on every metric over time.
{.is-success}
 
#### Manual
 
To fire a test at a single drive by hand:
 
```bash
smartctl -t short /dev/sda
smartctl -t long /dev/sda
```
 
The short test takes a couple of minutes. The long test can take hours on a big spinning drive and runs in the background while the drive stays in service.
 
### 3 · Read a SMART report
 
Find your disks first. This works for every drive type and tells you which are `scsi` (SATA/SAS) and which are `nvme`:
 
```bash
smartctl --scan
```
 
Then pull the data you need:
 
```bash
smartctl -a /dev/sda        # full health report
smartctl -H /dev/sda        # quick pass/fail summary
smartctl -l selftest /dev/sda   # results of the self-tests you ran
```
 
#### 3.1 The five metrics that matter
 
Backblaze found that about 77% of drives that died had already tripped a warning on one of these five. Watch the **raw** values and their trend over time, not the normalized score (which is proprietary and varies by manufacturer).
 
| Attribute | ID | What it flags | Healthy value |
|-----------|-----|---------------|---------------|
| Reallocated Sectors | 5 | Sectors remapped after going bad | 0, or low and stable |
| Current Pending Sectors | 197 | Bad sectors awaiting reallocation | 0 |
| Offline Uncorrectable | 198 | Sectors the drive cannot read | 0 |
| Reported Uncorrectable | 187 | Reads that ECC could not fix | 0 |
| Command Timeout | 188 | Aborted ops, often cabling or power | 0, or low and stable |
{.dense}
 
#### 3.2 Reading NVMe drives
 
Those five are ATA attributes, so they apply to spinning drives and SATA SSDs. NVMe drives do not have them. On an NVMe drive, watch these instead in the `smartctl -a` output:
 
- **Percentage Used**: write endurance consumed. 100% means the rated limit, not a dead drive.
- **Available Spare**: reserve area left for remapping bad blocks.
- **Media and Data Integrity Errors**: should stay at 0.
> 
> Many consumer NVMe SSDs do not support the self-test command at all. If a test errors out on an NVMe drive, that is expected. You still read its health with `smartctl -a`.
{.is-info}
 
### 4 · Extend drive life
 
#### 4.1 Schedule ZFS scrubs
 
Scrub scheduling is still in the GUI. Go to **Data Protection → Scrub Tasks** and make sure every pool has one. A scrub reads every block, verifies it against its checksum, and self-heals from redundancy if a block has gone bad. Schedule scrubs weekly or monthly during quiet hours.
 
#### 4.2 Do not overlap tests
 
Never run a long SMART test and a scrub (or resilver) on the same drive at the same time. Long self-tests can take the disk offline. Stagger their schedules so they never collide.

 
#### 4.3 Correlate before you condemn
 
When `zpool status` shows CKSUM, READ, or WRITE errors, cross-check SMART before you conclude:
 
- **SMART clean, ZFS errors climbing** → suspect the transport (cable, backplane, HBA, PSU). Reseat or replace the path before condemning the drive.
- **SMART shows reallocations, pending, or uncorrectables** → the drive is genuinely failing. Replace it.
#### 4.4 Keep them cool
 
Heat kills drives. Good airflow is one of the cheapest ways to extend drive life, and TrueNAS now monitors temperature on SATA and SAS disks. Fix airflow if a drive runs consistently hot.
 
#### 4.5 Have backups
 
Drives die and ZFS is not a magic bullet. There is no substitute for a real 3-2-1 backup: three copies, two different media, one off-site.

 
### 5 · SMART vs ZFS
 
They are two legs of the same table, and neither replaces the other. **SMART checks the container** (the physical drive, including empty sectors your data has not touched yet). **ZFS checks the contents** (your actual blocks, via checksums and scrubs, and self-heals them). ZFS knows whether your data is intact but can infer nothing about the health of the metal it lives on. SMART is the only thing that sees the hardware. Run both.
 
### <img src="/youtube.png" class="tab-icon"> 6 · Video

### Data Protection
https://youtu.be/bV7Y9jQrVPg

## Network
### Network

https://www.youtube.com/watch?v=0lzFHySymsU

### Adding a Static IP

Having a static IP to your server will prove necessary if you refer to your apps as IP:Port as shown in most of this wiki. If the IP of the server were to change, all of your apps would become unreachable.

1. Navigate to **Network** > **Interfaces** 
1. Click the **Pencil** icon next to your ethernet adapter
1. **Uncheck** the box for **DHCP**
1. Click **Add** for the **Aliases**. Enter your IP address, then select **/24**.

> Make sure the IP address you select is available
{.is-warning}


As long as you are accessing the WebGUI from the address you just selected, you should be given a prompt to **Save Changes**. 

> If you are coming from a different IP address, you will need to log back in within 60 seconds to the WebGUI at the new address you just chose, navigate to the **Network** page, then **Save Changes**.
{.is-warning}


### DNS Servers

We need to change the Nameservers away from our local to something more reliable. 
1. Navigate to **Network** > **Global Configuration** 
1. Click **Settings**
1. Change Nameserver 1 to `1.1.1.1` and Nameserver 2 to `9.9.9.9`.

### Building a Bridge

Before we setup the VM, we have to build a network bridge. This is necessary because without it, our VM won't be able to see anything on our TrueNAS host. Follow the [docs](https://www.truenas.com/docs/scale/scaletutorials/network/interfaces/settingupbridge/) or even better, follow this YouTube video:  
https://youtu.be/XBcAMd_wyI0

## Containers
## Virtual Machines
https://youtu.be/l1ISBq7cWkU

## Apps
### Apps

> Check out the new [TrueNAS Apps directory and guide](https://apps.truenas.com/)!
{.is-success}

### Per-App IP Addresses

1. Go to your router page and find an open IP in your subnet
1. Navigate to the **Network** Tab and edit the interface your server is running on
1. Add the open IP address as an additional **Alias**
1. Navigate to the app you want to assign the IP address to
1. Click **Edit** and scroll to the **Network Configuration** section, then the **Host IPs** subsection, then click **Add**
1. Select the new IP address and click **Save** at the bottom

> Note that the **WebUI** button will not open the correct IP when you click it; you must navigate to the app manually
{.is-warning}

https://youtu.be/n7V5e09w6uw

### Migrating Apps to Another Pool

> Always make sure you are using **host path configuration** for your volumes and take a backup of compose files first (if you are using them)!
{.is-warning}

[Follow this guide from Stux](https://forums.truenas.com/t/howto-copy-the-hidden-ix-apps-dataset-from-one-pool-to-another/24434)

## System
### General Settings
#### Managing Your Configuration File

You should be taking regular backups of your Configuration File. Especially when upgrading versions, which you will be prompted to do before you start the upgrade.

To take a backup, click **Manage Configuration → Download File**. In the event you ever need to restore from that file, click **Upload File**.
> 
> When uploading a config file it will overwrite your current settings with the saved ones from the file!
{.is-warning}

#### GUI
Some apps like Nginx Proxy Manager require ports `80` and `443`. By default, TrueNAS uses these for the webGUI. Change them by clicking the **Settings** button to any other open port.

After you save the changes you will lose connection to the UI and have to navigate to the new port in the URL.



# <img src="/youtube.png" class="tab-icon"> Hardening

[Lawrence](https://www.youtube.com/@LAWRENCESYSTEMS) does a great job of going over some basic security hardening tasks. Watch his video for some tips.  
https://www.youtube.com/watch?v=fEWobaEIAHM