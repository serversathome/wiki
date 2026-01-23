---
title: Gotify
description: A guide to installing Gotify in docker via compose
published: true
date: 2026-01-23T17:38:46.725Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:55.681Z
---

# ![](/gotify.png){class="tab-icon"} What is Gotify?

Gotify is a free and open source project that lets you control your data and communicate via a REST-API and a web socket.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Gotify

```yaml
services:
 server:
   ports:
     - 8001:80
   restart: unless-stopped
   container_name: gotify
   volumes:
     - /mnt/tank/configs/gotify:/app/data
   image: gotify/server
```

# 2 · Logging In

1. The default credentials are `admin:admin`
1. Once you login, click the name **ADMIN** on the purple bar at the top to change the password.

# 3 · Apps

To create a way for your services to talk to you, each service should be its own “app”. Go to the APPS section on the header and make one app per service. 

![](/screenshot_from_2024-08-30_15-47-57.png)

On the client side, it will ask you for your server IP or URL and the app or API token, which you can copy and paste after revealing it by clicking the eye incon shown above.

> You'll notice my apps have nice icons which represent the service they are named for. To change the icon, click the cloud logo next to it and upload a saved image. You can find icons like the ones I have here at [https://selfh.st/icons/](https://selfh.st/icons/)
{.is-info}


# <img src="/emby.png" class="tab-icon"> 4 · Emby

Emby needs some special steps to use Gotify which other apps do not. Steps are below (copied from [here](https://emby.media/community/index.php?/topic/90463-gotify-notifications/)):

1.  Download the .dll plugin [here](https://github.com/rootforbid/Emby.Plugins.Gotify/releases/download/v2.0.0.0/MediaBrowser.Plugins.GotifyNotifications.dll).
2.  Put it in the "plugins" folder of your Emby Server and restart Emby Server.
3.  The plugin should now show up in your Emby Server management page under Advanced > Plugins.
4.  Configure the plugin by heading to the new Notification section under User settings.
5.  Click the "+ Add Notification" button and choose "Gotify"

# <img src="/truenas.png" class="tab-icon"> 5 · TrueNAS

We need to deploy a custom app for this to work. Use the YAML file below, entering in your IP address and token for Gotify:

```yaml
services:
 gotify-truenas:
   environment:
     - GOTIFY_URL=
     - GOTIFY_TOKEN=
   image: ghcr.io/ztube/truenas-gotify-adapter:main
   network_mode: host
   restart: unless-stopped
```

Next, navigate to the **Alert Settings** page and add this alert for Slack:

![](/image.png)

| Field | Value |
| --- | --- |
|Name|Gotify|
|Type|Slack|
|Level|Warning|
|Webhook URL|http://localhost:31662|

# <img src="/youtube.png" class="tab-icon"> 6 · Video
https://youtu.be/CaKs9M2SL3k