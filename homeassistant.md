---
title: Home Assistant
description: A guide to deploying Home Assistant on TrueNAS
published: true
date: 2025-06-15T11:47:32.588Z
tags: 
editor: markdown
dateCreated: 2025-06-12T14:57:25.253Z
---

![homeassistant.png](/homeassistant.png)

# What is Home Assistant?
Home Assistant is free and open-source software used for home automation. It serves as an integration platform and smart home hub, allowing users to control smart home devices. 

# Installation

## VM

### Creating the VM on TrueNAS
1. Go to the [official Ubuntu Desktop download page](https://ubuntu.com/download/desktop) to grab the latest ISO of Ubuntu Desktop.
1. Create a new Instance in TrueNAS, give it a name and select VM under the **"Virtualization Method"**. Under **"VM Image Options"** select the **"Upload ISO"** radio button. Click the **"Select Volume"** button and upload the Ubuntu Desktop ISO you downloaded in step 1.

	![Screenshot_2025-06-12-081310.png](/Screenshot_2025-06-12-081310.png)

	![Screenshot_2025-06-12-082603.png](/Screenshot_2025-06-12-082603.png)

1. Give the VM an appropriate amount of resources for Home Assistant. This can vary depending on how many add-ons and devices you plan on running. Don't worry about resources for Ubuntu, we are only using the bootable .iso to download and install the HAOS (home assistant operating system). I would recommend 2 CPUs and 8 GiB of ram to start. You can always increase this later. For storage I recommend at least 32GiB to start. For the **"Root Disk I/O Bus"** you need to select which works best for your system.

	![Screenshot_2025-06-12-083558.png](/Screenshot_2025-06-12-083558.png)

	![Screenshot_2025-06-12-084257.png](/Screenshot_2025-06-12-084257.png)

1. For network I chose my previously made bridge "br0".
1. Enable VNC and give it a password of your choosing. If you already have a VM with VNC enabled you will need to change the default 5900 port. I chose 5901. 
1. Create the VM. 


### Inside the VM

1. VNC into the VM and select "Try or Install Ubuntu". Follow the on screen prompts, **skip the update**. Once you see two options, install or try, select **Try Ubuntu** and click close.

	![Screenshot_2025-06-12-085240.png](/Screenshot_2025-06-12-085240.png)

	![Screenshot_2025-06-12-090034.png](/Screenshot_2025-06-12-090034.png)


1. Open Firefox in your Ubuntu VM and proceed to the [official Home Assistant Installation guide](https://www.home-assistant.io/installation/generic-x86-64). Next, jump to the section titled: **Method 1: Installing HAOS via Ubuntu booting from a USB flash drive**. Skip to step 5 and click the "download the image" link. Once the download is complete, skip to step 7 and following the instructions. Select the disk you sized during the creation of the VM in TrueNAS.

1. Once the restore process is done, power off the Ubuntu VM and remove the .iso from the Disks section in TrueNAS. Start the VM and VNC back into it. You should see HAOS booting and eventually you will see the splash screen.

1. You will need to find the IP address of the VM via your router and use the port 8123. Home Assistant will default to giving you homeassistant.local:8123 address. Once you have the IP you may proceed to the [Onboarding Process](https://www.home-assistant.io/getting-started/onboarding/).


## LXC

It is also possible to use [Helper Scripts](https://bketelsen.github.io/IncusScripts/scripts?id=homeassistant) to install Home Assistant as an LXC. Use the command below ([after installing the `scripts-cli`](https://wiki.serversatho.me/en/TrueNAS#incus-helper-scripts)) to launch the container:

```bash
scripts-cli launch homeassistant homeassistant
```

### USB Passthru

To pass devices into the container, click the name of the instance and using the **Devices** menu on the right.
![screenshot_from_2025-06-15_07-44-45.png](/screenshot_from_2025-06-15_07-44-45.png)

> Note this this is untested for USB passthru as of 6/15/25
{.is-warning}

