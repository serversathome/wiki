---
title: Ghost
description: A guide to installing Ghost in docker via compose
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:44.549Z
---

# <img src="/ghost-light.png" class="tab-icon"> What is Ghost?

Ghost is a powerful app for professional publishers to create, share, and grow a business around their content. It comes with modern tools to build a website, publish content, send newsletters & offer paid subscriptions to members.

<div class="glance">
  <div><span>Port</span><b><code>8085</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>MySQL</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://ghost.org/docs/config/"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://youtu.be/YnjYWhceepU?feature=shared&t=227"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Ghost

```yaml
services:

 ghost:
   image: ghost:6-alpine
   container_name: ghost
   restart: unless-stopped
   ports:
     - 8085:2368
   environment:
     database__client: mysql
     database__connection__host: db
     database__connection__user: ghost
     database__connection__password: your-db-password
     database__connection__database: ghost
     url: https://example.com # this url value is just an example, and is likely wrong for your environment!
     TZ: America/New_York
     security__staffDeviceVerification: false
     privacy__useTinfoil: true
     mail__options__auth__pass: 
     mail__options__auth__user: 
     mail__options__host: 
     mail__options__port: 587
     mail__options__secure: false
     mail__options__service: 
     mail__transport: SMTP
     mail__from: 
   volumes:
     - /mnt/tank/configs/ghost/content:/var/lib/ghost/content

 db:
   image: mysql:8.0.44
   container_name: ghost_db
   restart: unless-stopped
   environment:
     MYSQL_ROOT_PASSWORD: your-root-password
     MYSQL_USER: ghost
     MYSQL_PASSWORD: your-db-password
     MYSQL_DATABASE: ghost
   volumes:
     - /mnt/tank/configs/ghost/db:/var/lib/mysql
```

1. Change all the default passwords (generate them with `openssl rand -hex 24`). `database__connection__password` in the ghost service must match `MYSQL_PASSWORD` in the db service.
1. Update the `url` field to match your FQDN

> You can deploy as many of these containers as you want for each website you want to host. Be sure to change the external port for each one. 
{.is-info}

> **Upgrading from an older version of this guide?** It used `ghost:5-alpine`, which no longer gets updates. Before switching to `ghost:6-alpine`, back up `/mnt/tank/configs/ghost` and read Ghost's [update guide](https://docs.ghost.org/update/) and the [Ghost 6.0 changelog](https://ghost.org/changelog/6/). The `MYSQL_USER`/`MYSQL_PASSWORD` user is only created on a fresh database, so if your existing database was set up with `root`, keep `database__connection__user: root` and your old password.
{.is-warning}

> **Existing installs:** older versions of this guide mounted Ghost's content at `/mnt/tank/configs/ghost` with the database nested inside it at `/mnt/tank/configs/ghost/db`. The compose above keeps them apart (`ghost/content` and `ghost/db`) so Ghost's ownership changes can't touch MySQL's files. If you deployed the old layout, either keep your original `/mnt/tank/configs/ghost:/var/lib/ghost/content` line, or stop the stack and move everything except `db` into a new `content` folder before redeploying.
{.is-info}


# 2 · Logging In

Once you have deployed this container go to https://example.com/ghost to login and create your account.

# 3 · Monitoring With Umami

In order to enable monitoring, go to the **Settings** gear icon in the lower left corner, then select **Code Injection** section (under **Advanced**), and in the **Site Header** section paste the Tracking Code from the website you created in Umami.

# 4 · Getting Mail Working

You don't need mail to work to host content on Ghost. If you want to have subscribers sign up and get emails when you update content however, you will need email set up. You will also need email configured if you want to have multiple contributors with individual accounts on your platform. Mailgun's free plan includes 100 emails per day.

## 4.1 Mailgun
> 
> If you go this route remove any lines in the docker compose file which start with `mail`
{.is-danger}


Navigate to [mailgun](https://signup.mailgun.com/new/signup). Enter all the information keeping the default **Foundation** **50k Trial** selected (it's free for the first month, then becomes a paid plan unless you downgrade to the free plan). Once you have an account, login to your account on mailgun and follow the steps in [this video](https://youtu.be/YnjYWhceepU?feature=shared&t=227) on the correct way to get email configured.

## 4.2 Transactional Email

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

# 5 · Migrating From Wordpress

> In the event you want to move all your content from Wordpress, there are a few steps to take. Official instructions can be found [here](https://ghost.org/docs/migration/wordpress/). 
{.is-info}


## 5.1 Posts and Pages

Install the [Ghost Export Plugin](https://wordpress.org/plugins/ghost/) to your WP instance. I could not get the full .zip with all the images and such to work, but if you can, all you need to do is export your site as a zip file and upload it to Ghost (Ghost > Settings > Import/Export > Universal Import). For those who can't get the full .zip working, use the JSON instead, knowing it won't contain any of your pictures/media.

## 5.2 Redirects

If you left the WP settings for the posts default, it uses a directory tree which includes the /year/month/date/post-name format. Ghost does not use that, so you need to set up 301 redirects in Ghost. Create a .yaml file with these contents *keeping the spacing exactly as you see it*:

```yaml
301:
  ^\/(\d{4})\/(\d{2})\/(\d{2})\/(.*)$: /$4
```

After that, upload this in the Labs section (Ghost > Settings > Labs > Redirects).