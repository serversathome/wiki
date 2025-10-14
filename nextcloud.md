---
title: Nextcloud
description: A guide to deploying Nextcloud on TrueNAS Scale and via docker compose
published: true
date: 2025-10-14T09:58:21.436Z
tags: 
editor: markdown
dateCreated: 2024-11-06T16:25:41.034Z
---

# ![](/nextcloud.png){class="tab-icon"} What is Nextcloud?

Nextcloud is a self-hosted cloud file storage and collaboration software that offers productivity, control and compliance for any organization.

# 1 · Deploy Nextcloud
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
 nextcloud:
   image: lscr.io/linuxserver/nextcloud:latest
   container_name: nextcloud
   environment:
     - PUID=568
     - PGID=568
     - TZ=America/New_York
   volumes:
     - /mnt/tank/configs/nextcloud/config:/config
     - /mnt/tank/configs/nextcloud/data:/data
   ports:
     - 8887:443
   restart: unless-stopped
```

I have changed the **PUID** and **PGID** to the TrueNAS `apps` user and group. I have also changed the external port in to one which is less likely to have anything running on it. 

## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](/screenshot_from_2024-11-06_11-06-46.png)

1. Under the **Nextcloud Configuration** add the **ffmpeg, smbclinet, and ocrmypdf** packages
1. Add a **Tesseract Language Codes** set to **eng** (if you speak english)
1. When doing the deployment, make sure to use secure passwords for the **Admin User**, **Admin Password**, **Redis Password**, and **Database Password**

1. The **Host** should be the FQDN you plan to use (like nextcloud.example.com)

1. For the **Storage Configuration** make either datasets or subdirectories for the AppData Storage, User Data Storage and Postgres Data Storage. They all must be in separate directories. 

> When setting the hostpath for the Postgres dataset make sure to check the box for **Automatic Permissions** or the app won't launch!
{.is-danger}


![](/screenshot_from_2024-11-28_21-24-00.png)

> If you want to access directories on your NAS, be sure to add **Additional Storage** so Nextcloud can see it
{.is-info}

> Check out [the new docs](https://apps.truenas.com/resources/deploy-nextcloud) from TrueNAS
{.is-success}

# 2 · Post-Install

Nextcloud will have some warnings from certain variables not being set out-of-the-box. Shell into Nextcloud, run this command, then **restart** your container:

```bash
occ config:system:set maintenance_window_start --value=3 && \
occ maintenance:repair --include-expensive && \
echo 'Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains; preload"' >> /etc/apache2/conf-available/security.conf && \
a2enconf security && \
service apache2 restart && \
occ db:add-missing-indices && \
occ config:system:set default_phone_region --value="XX" && \
occ config:system:set mail_smtpmode --type=string --value="log"
```


# 3 · Nextcloud Configuration

## 3.1 Accessing Your Container

You will not be able to go to the private IP of your Nextcloud instance because you need an https connection for it to work. This may work if you set a **Certificate ID** in TrueNAS during install, but it won't matter because we need to adjust the **Trusted Domain** array no matter what. 

To do this, go the the directory where the configs are stored (usually the mount path followed by /var/www/html/config/config.php). Edit this file and find the **Trusted Domains Array** and add another line at the bottom for your FQDN. Make sure it lines up underneath the other entries as spacing matters. Your line should look like `4 => ‘nextcloud.example.com’,` incrementing to whatever number would be next in your array. Once this is done restart the container.

## 3.2 Adding External Storage

The point of Nextcloud is to be able to access your files remotely. To do this, you need to add **Additional Storage** from TrueNAS or another volume in the compose file so Nextcloud can see your files. Once you have that added, navigate within Nextcloud to **Apps** (top right circle) → **Disabled Apps** → **External Storage** and **Enable** that app.

Once that is enabled navigate to **Administration Settings** → **External Storage** (the one at the bottom under the **Administration** heading). Give it a name in the leftmost box, then in the **Configuration** field enter the path you mounted it as (so if I added a volume which was `/mnt/tank/files:/files` I would use the internal path here of `/files`). Add a person or group that this will be available to and click the check box at the end of the row. I recommend you also click the 3 dot box at the end of the row and select the checkbox to **Enable Sharing** so you can give people links to your files if you want.

## 3.3 Logging in With Your Phone

To access the fastest way via mobile, use a QR code to login. To generate the QR code:
1. Navigate to **Administration Setings → Security**
1. At the bottom, add an **App name** then click the blue button next to it to **Create a new app password**
1. Click the button to **Show QR code for mobile apps**

# 4 · Deploy Collabora

To use Nextcloud as a collaborative documentation platform, we need to add Collabora in a separate container and then create a reverse proxy record for it. First deploy another container using the docker compose file.

## <img src="/docker.png" class="tab-icon"> 4.1 Docker Compose

```yaml
services:
  collabora:
    image: collabora/code
    container_name: collabora
    restart: unless-stopped
    ports:
      - 9980:9980
    environment:
      - TZ=America/New_York
      - domain=office\\.example\\.com
      - server_name=office.example.com
      - username=admin
      - password=adminpassword
    volumes:
      - /mnt/tank/configs/collabora:/data
```


Note for the `domain` it needs to be in regex. Replace what I have with your correct FQDN values.

## 4.2 Reverse Proxy

We now need a reverse proxy entry for something like *office.example.com* pointed to an **https** entry for our sever IP. If you use Cloudflare tunnels it will look like this:

![](/screenshot_from_2025-01-17_10-42-55.png)

Note that since Collabora has its own self-signed certificate we need to check the **No TLS Verify** option with Cloudflare or the tunnel will not connect. 

## 4.3 Nextcloud Settings

Now navigate to **Nextcloud** > **Apps** > **Office & text** \> **Nextcloud Office** and enable/install this app. 

Next navigate to **Nextcloud** > **Administration Settings** >  **Office** and click the radio button to **Use your own server**. Enter the address of your FQDN and click save. You should see a green bar across the top that says *Collabora Online server is reachable*.

https://youtu.be/ibL9qAlUZes

# <img src="/youtube.png" class="tab-icon"> YouTube Walkthrough

[https://www.youtube.com/watch?v=pAebJIDT\_oc](https://www.youtube.com/watch?v=pAebJIDT_oc)

[https://youtu.be/qn5ccoCabdA](https://youtu.be/qn5ccoCabdA)