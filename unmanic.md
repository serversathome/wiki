---
title: Unmanic
description: A guide to deploying Unmanic
published: true
date: 2025-08-14T20:54:01.141Z
tags: 
editor: markdown
dateCreated: 2025-08-14T19:02:47.616Z
---

# ![](/unmanic.png){class="tab-icon"} What is Unmanic?

Unmanic gives you the power to automate the management of any file library through the use of customised modular task flows to suit your specific needs, giving you the ultimate, simple to configure, set-and-forget library optimisation tool.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Unmanic

```yaml
services:
  unmanic:
    container_name: unmanic
    restart: unless-stopped
    image: josh5/unmanic:latest
    runtime: nvidia
    ports:
      - 8870:8888
    environment:
      - PUID=568
      - PGID=568
      - NVIDIA_VISIBLE_DEVICES=all
    volumes:
      - /mnt/tank/configs/unmanic:/config
      - /mnt/tank/media/:/library
      - ./temp:/tmp/unmanic
```

# 2 · Unmanic Configuration
1. Upon first login, expand the left pane with the hamburger menu ☰ in the top left
1. Click on **Settings**

## 2.1 Plugins
1. Click on **Plugins**
1. Click **Install Plugin From Repo**
	a. Click **Refresh Repositories**
	b. In the search bar type `transcode`
	c. Click the purple button in the **Transcode Video Files** box to download the plugin
	d. Close the window
1. Click the **Settings** button in the new Transcode Video Files box
	a. Switch the **Video Encoder** to `NVENC` if you are using an nVidia GPU

> There are **many** other plugins that do various functions. Take time to explore more!
{.is-info}

## 2.2 Workers
1. Click on **Workers**
1. Click on the + symbol in the **Worker Groups** section
1. Give it a name
1. Increase your worker count based on what your GPU can handle


## 2.3 Libraries
1. Click on **Library**
	a. Under **Libraries** click on the **Settings** button
	b. Set the **Library Path** to your movies directory
	c. Turn on the switch for **Enable library scanner for this library**
	d. Turn on the switch for **Enable file monitor for this library**
	e. Click the + icon in the **Plugins** section and select the Transcode Plugin
1. Click the + Icon under **Libraries** to add a new library
	a. Click the folder icon at the top with the two periods ( .. ) to go up one path
 	b. Set your **Library Path** to your TV directory
 	c. Click the arrow button at the top right corner to close the window
	d. Follow the same stop above to turn on scanning and monitoring

> You may get a warning at the bottom of the screen which says `Unmanic has stopped all workers..` This is normal
{.is-info}

## 2.4 Start Workers
1. Click the **Hom** button at the top
1. In the **Pending Tasks** box, click **Rescan Library Now**
1. You should see your workers spin up as they find media to transcode to h265

# 3 · Recommended Plugins

| Name | Function |
|----|-----|
|File Size Metrics Data Panel | Aggregate changes in file size metrics for processed files and add a data panel for displaying the results. |
| Modify Unmanic's Default File Movements | Good if you don't want to delete the original file. |
| Notify Radarr/Sonarr | Notify Radarr/Sonarr on movie or series update to rescan and rename a modified file. |
| Reject File if Larger than Original | This plugin will reset the current working file to the original source if it is larger than the original source. | 


