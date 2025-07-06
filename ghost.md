---
title: Ghost
description: A guide to installing Ghost in docker via compose
published: true
date: 2025-07-06T10:27:46.357Z
tags: 
editor: markdown
dateCreated: 2024-08-11T11:43:39.558Z
---

![ghost-light.png](/ghost-light.png)

# What is Ghost?

Ghost is a powerful app for professional publishers to create, share, and grow a business around their content. It comes with modern tools to build a website, publish content, send newsletters & offer paid subscriptions to members.

# Docker Compose

```yaml
services:

 ghost:
   image: ghost:5-alpine
   container_name: ghost
   restart: unless-stopped
   ports:
     - 8085:2368
   environment:
     # see https://ghost.org/docs/config/#configuration-options
     database__client: mysql
     database__connection__host: db
     database__connection__user: root
     database__connection__password: example
     database__connection__database: ghost
     url: https://example.com # this url value is just an example, and is likely wrong for your environment!
     TZ: America/New_York
     security__staffDeviceVerification: false
     privacy_useTinfoil: true
   volumes:
     - /mnt/tank/configs/ghost:/var/lib/ghost/content

 db:
   image: mysql:8.0
   container_name: ghost_db
   restart: unless-stopped
   environment:
     MYSQL_ROOT_PASSWORD: example
   volumes:
     - /mnt/tank/configs/ghost/db:/var/lib/mysql
```

1. Change all the default user names and passwords. Make sure when you change them in the ghost service you mirror those changes in the db service.
1. Update the `url` field to match your FQDN

> You can deploy as many of these containers as you want for each website you want to host. Be sure to change the external port for each one. 
{.is-info}


# Logging In

Once you have deployed this container go to https://example.com/ghost to login and create your account.

# Monitoring With Umami

In order to enable monitoring, go to the **Settings** gear icon in the lower left corner, then select **Code Injection** section (under **Advanced**), and in the **Site Header** section paste the Tracking Code from the website you created in Umami.

# Getting Mail Working

You don't need mail to work to host content on Ghost. If you want to have subscribers sign up and get emails when you update content however, you will need email set up. You will also need email configured if you want to have multiple contributors with individual accounts on your platform. Mailgun is free as long as you send less than 1000 emails per month.

## Mailgun

Navigate to [mailgun](https://signup.mailgun.com/new/signup). Enter all the information keeping the default **Foundation** **50k Trial** selected. Once you have an account, login to your account on mailgun and follow the steps in [this video](https://youtu.be/YnjYWhceepU?feature=shared&t=227) on the correct way to get email configured.

## Transactional Email

Once you get to the 10 min mark in the video, he switches gears from bulk email to transactional email. Because we deployed in docker, we need to get access to our container a different way than in the video. *(If you can get to the console of the container, skip to step #3)* First we need to ssh into our server with sudo privileges. Once there, follow these steps to get the container ID and then execute the command shown:

1.  **Find the Container ID or Name:** You need to know the container ID or name of the container you want to access. You can list all running containers with: `docker ps`
2.  **Execute a Command or Start a Shell:** To start a shell session inside the container, use: `docker exec -it <container_id_or_name> /bin/bash`
3.  After you execute that command, we will be in the directory containing `config.production.json`. To edit it, enter the command `nano config.production.json` and follow the steps in the video to replace the “mail” block with the [example from Ghost](https://ghost.org/docs/config/#mail) with the correct SMTP credentials from mailgun:

```json
// config.production.json

"mail": {
  "transport": "SMTP",
  "options": {
    "service": "Mailgun",
    "auth": {
      "user": "postmaster@example.mailgun.org",
      "pass": "1234567890"
    }
  }
},
```

  4. Now restart your container and you're done!

# Migrating From Wordpress

> In the event you want to move all your content from Wordpress, there are a few steps to take. Official instructions can be found [here](https://ghost.org/docs/migration/wordpress/). 
{.is-info}


## Posts and Pages

Install the [Ghost Export Plugin](https://wordpress.org/plugins/ghost/) to your WP instance. I could not get the full .zip with all the images and such to work, but if you can, all you need to do is export your site as a zip file and upload it to Ghost (Ghost > Settings > Import/Export > Universal Import). For those who can't get the full .zip working, use the JSON instead, knowing it won't contain any of your pictures/media.

## Redirects

If you left the WP settings for the posts default, it uses a directory tree which includes the /year/month/date/post-name format. Ghost does not use that, so you need to set up 301 redirects in Ghost. Create a .yaml file with these contents *keeping the spacing exactly as you see it*:

```yaml
301:
  ^\/(\d{4})\/(\d{2})\/(\d{2})\/(.*)$: /$4
```

After that, upload this in the Labs section (Ghost > Settings > Labs > Redirects).