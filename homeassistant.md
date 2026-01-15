---
title: Home Assistant
description: A guide to deploying Home Assistant on TrueNAS
published: true
date: 2026-01-15T15:29:33.794Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:16.650Z
---

# ![](/homeassistant.png){class="tab-icon"} What is Home Assistant?
Home Assistant is free and open-source software used for home automation. It serves as an integration platform and smart home hub, allowing users to control smart home devices. 

# ![](/homeassistant/homeassistant.png){class="tab-icon"} What is Home Assistant?
Home Assistant is free and open-source software used for home automation. It serves as an integration platform and smart home hub, allowing users to control smart home devices.

# 1 · Prerequisites
To install Home Assistant, you need a couple of things:
- An Ubuntu based ISO file (Ubuntu Desktop works fine, but I had better luck with Linux Mint)
- Minimum of 2 CPU cores (more is better)
- Minimum of 8GB memory (more is better)
- A network bridge set up

# 2 · Setup VM

### 2.1 Operating system
Set the options as follows:

Guest Operating System = Linux
Name = (whatever you want)
Password = (password for the VNC)

Leave everything else alone

<img width="449" height="985" alt="image" src="https://github.com/user-attachments/assets/e1cd6220-2a6e-4de4-8acb-d7e4436b72df" />

### 2.2 CPU And Memory
Set the options as follows:

Virtual CPUs = 1 (it's important to leave this at 1 unless you have dual sockets)
Cores = 2 (or more)
Threads = 2 (or more)
CPU Mode = host
Memory size = 8 GiB (or more)

Leave everything else alone

<img width="459" height="973" alt="image" src="https://github.com/user-attachments/assets/662be758-75d2-44e7-a1fe-ae9272dfe4f9" />

### 2.3 Disks
Specify a zvol location and give it a minimum of 32 GB

<img width="438" height="469" alt="image" src="https://github.com/user-attachments/assets/6ff94310-0031-4469-ac6a-23d779061e81" />

### 2.4 Network
Choose the network adapter type that you have available and attach it to the NIC (br0 if you have a bridge)

<img width="434" height="381" alt="image" src="https://github.com/user-attachments/assets/a21c0dde-503f-4bce-8089-4973c2fc288e" />

### 2.5 Installation Media
Point to the Ubuntu Desktop/Linux Mint ISO file (or upload it if you need to)
Leave everything else alone and click

# 3. Installing Home Assistant

1. VNC into the VM and select "Try or Install Ubuntu (or Start Linux Mint)".

<img width="925" height="562" alt="image" src="https://github.com/user-attachments/assets/342107c5-6ebf-4505-80e1-099d7ed43017" />

2. Open Firefox in your VM and proceed to the [official Home Assistant Installation guide](https://www.home-assistant.io/installation/generic-x86-64). Next, jump to the section titled: **Method 1: Installing HAOS via Ubuntu booting from a USB flash drive**. Skip to step 5 and click the "download the image" link. Once the download is complete, right click on the file and open with **Disk Image Writer**. Set the destination to the disk you added during the VM setup and click **Start Restoring**

<img width="799" height="244" alt="image" src="https://github.com/user-attachments/assets/7985a772-c87a-4a75-8afd-31fde625698e" />

<img width="519" height="255" alt="image" src="https://github.com/user-attachments/assets/24df6c0f-fff6-4f0f-a9fc-7beff185911a" />

3. Once the restore process is done, power off the VM and remove the .iso from the Disks section in TrueNAS. Start the VM and VNC back into it. You should see HAOS booting and eventually you will see the splash screen.

4. You will need to find the IP address of the VM via your router and use the port 8123. Home Assistant will default to giving you homeassistant.local:8123 address. Once you have the IP you may proceed to the [Onboarding Process](https://www.home-assistant.io/getting-started/onboarding/).

