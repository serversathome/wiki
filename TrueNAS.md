---
title: TrueNAS Community Edition
description: This article will describe how to set up a TrueNAS server to be compatible will services described in this wiki.
published: true
date: 2025-06-16T22:11:11.434Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:25:40.008Z
---

![](/screenshot_from_2024-10-21_16-12-44.png)

> This page was built to describe TrueNAS CE Fangtooth 25.04.1
{.is-info}

# Installation
[](https://youtu.be/cA8fZ-lfgaA?feature=shared)


# YouTube Basic Walkthrough

See the playlist here: https://youtube.com/playlist?list=PL6zQmF2gDqDT7SHyBe7ni1P2S4NzyJpD6&feature=shared

# {.tabset}

## Dashboard
### Dashboard
[](https://youtu.be/wbeAWq8WiqE?feature=shared)

## Storage
### Storage
[](https://youtu.be/YgTFRrwJnwY?feature=shared)

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

[https://youtu.be/mm3lyPQcseE](https://youtu.be/mm3lyPQcseE)

### Adding a Single Disk

To [add a single disk](https://www.truenas.com/docs/scale/scaletutorials/storage/managepoolsscale/#extending-a-raidz-vdev) to an existing RAIDZ(1,2,3) pool, use the **Extend** button in the devices menu.

![devicesmirrorvdevselected.png](/devicesmirrorvdevselected.png)

Note that you will **not** be able to gain 100% of the usable space from that disk until you run the [rebalancing script](https://github.com/markusressel/zfs-inplace-rebalancing). You can calculate how much capacity you will gain by using the [calculator](https://www.truenas.com/docs/references/extensioncalculator/).

To view progress of the expansion, run this command in the shell: 
```bash
zpool status pool_name
```
Watch Lawrence do it:
[](https://youtu.be/uPCrDmjWV_I?feature=shared)


## Datasets
### Datsets
[](https://youtu.be/27MhvLKBKtQ?feature=shared)

For snapshots and rollbacks at a granular level, datasets should be set up in the pool for each individual app.

![](/screenshot_from_2025-01-20_10-50-23.png)

### Host Path Config Storage

Apps running on Scale should use the Host Path Config option to store their data. You will find this option in the right hand menu under **Storage Configuration** > *Type*. This allows for the rapid redeployment of apps with no loss to their configurations, which can be considerable for some. Each app uses its respective host path from the example above. For a video explanation and example of this:  
[https://youtu.be/JZ9zbcyLcDo](https://youtu.be/JZ9zbcyLcDo)

### Permissions

In order for the apps to write to the dataset properly, make sure to select the ‘Apps’ option for the dataset preset:

![](/screenshot_from_2024-10-21_16-10-36.png)

Once the dataset has been created, you can still modify its permissions like below:

![](/screenshot_from_2024-02-23_11-52-29.png)

For a video walkthrough on permissions:  
[https://youtu.be/qAGN0\_73cV4](https://youtu.be/qAGN0_73cV4)

## Shares
### Shares
[](https://youtu.be/RAlJ3fcktMQ)

## Data Protection
### Data Protection
[](https://youtu.be/bV7Y9jQrVPg)

## Network
### Network

[](https://www.youtube.com/watch?v=0lzFHySymsU)

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
[](https://youtu.be/XBcAMd_wyI0)

## Instances
### Instances
[](https://youtu.be/z2UUBW1eHsk)

### Upgrading VMs From Previous Versions

Right now this is difficult to do with Windows but has been fairly straight forward with Linux. With Linux I notice the ethernet device changes upon migration, so if you are having issues use VNC to look at the console and troubleshoot your networking.

For Windows [this migration guide](https://forums.truenas.com/t/guide-how-to-install-migrate-windows-vm-to-fangtooth-incus-using-virtio-drivers/37056) should help. I didn't write it, but it seems to work for many people.

### Changing Default Port 53

By default, Incus runs on port 53, which will conflict with any DNS servers you want to run. To change the default port for Incus, run this command in the TrueNAS shell:

```bash
sudo incus network set incusbr0 raw.dnsmasq="port=5354"
```

Also, if you use a bridge and set Incus to use the bridge, it also solves the conflict.

### Incus Helper Scripts

> Note that this repo is under development so not everything is functional!
{.is-warning}

Starting on Fangtooth, the ability to launch LXCs via the TrueNAS command line is now a thing. Similar to Proxmox, the creation of pre-built containers is vastly simplified by the use of scripts. [See this website](https://bketelsen.github.io/IncusScripts/) for a list of scripts available.

#### Installing the CLI

In order to use the scripts you must first install the Helper Scripts CLI.
1. Create a dataset for the CLI (unmodified general permissions will be fine)
1. Navigate to the TrueNAS shell
1. Enter these commands, assuming the path of the dataset is `/mnt/tank/scripts`:
```bash
cd /mnt/tank/scripts
wget https://github.com/bketelsen/IncusScripts/releases/download/v0.2.1/scripts-cli_Linux_x86_64.tar.gz
tar -xzvf scripts-cli_Linux_x86_64.tar.gz
echo 'export PATH=$PATH:/mnt/tank/scripts' >> ~/.zshrc
source ~/.zshrc
echo 'export PATH=$PATH:/mnt/tank/scripts' >> ~/.bashrc
source ~/.bashrc
```

#### Uninstalling the CLI

In the event you want to remove the scripts, simple delete the dataset you downloaded the to CLI in the above step. If you are still having issues, trying running this command in the TrueNAS shell:

```bash
incus profile rm scriptcli-storage
```

[](https://youtu.be/O9HeWPosBUQ)

### Creating a Windows VM

1. Download the .iso file for the version of Windows you want. These .iso files were compiled in May 2025 with the virtio drivers built in:

    - [Windows 10 Pro *OOBE Enabled by default*](http://serversatho.me/win10mod.iso)
    - [Windows 11 *OOBE Enabled by default*](http://serversatho.me/win11mod.iso)
    - [VirtIO Drivers](http://serversatho.me/virtio-win.iso)
    {.links-list}

1. Create a New Instance
1. Give it a name
1. Click **VM** and select the radio button for **Upload ISO, import a zvol or use another volume**
1. Click **Upload ISO** and select the ISO you just downloaded
1. You should see it mounted in the Volumes box and can click **Select**
1. Assign CPU and Memory
1. In **Storage** select **Virtio-SCSI** as the *Root Disk I/O Bus*
1. Give it 60 GiB or more *Root Disk Size*
1. Click the box to **Enable VNC**
1. Click **Create**
1. You now must jump to the VNC **quickly** so you can press any key to boot from the CD!!
1. Go through the install per usual
1. Once you have successfully booted into Windows, shutdown the VM
1. Add the **VirtIO Drivers** ISO from above to the **Disks** with a *Boot Priority* of `2` and an *I/O Bus* of `Virtio-SCSI`
1. Start the VM
1. Navigate to **Device Manager** and right-click the **Ethernet Device**
1. Click **Update Driver**
1. Click **Browse my Computer for Drivers** then click **Browse**
1. Select the drive which has the VirtIO disk mounted in it
1. Click **Next** then **Finish**
1. To enable RDP, type `settings` into the Start Bar
1. Search for `remote desktop`
1. Click **Remote Desktop Settings** and the flip the switch to **Enable Remote Desktop**

> Activation information can be found [here](https://massgrave.dev/)
{.is-info}

[]()


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

[](https://youtu.be/n7V5e09w6uw)


# Hardening

[Lawrence](https://www.youtube.com/@LAWRENCESYSTEMS) does a great job of going over some basic security hardening tasks. Watch his video for some tips.  
[https://youtu.be/u0btB6IkkEk?feature=shared](https://youtu.be/u0btB6IkkEk?feature=shared)