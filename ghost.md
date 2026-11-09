---
title: Ghost
description: A guide to installing Ghost in docker via compose
published: true
date: '2026-11-09T03:21:02.939Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:44.549Z'
---

# What is Ghost?

Ghost is a powerful app for professional publishers to create, share, and grow a business around their content. It comes with modern tools to build a website, publish content, send newsletters & offer paid subscriptions to members.

# :logos:docker-icon: 1 · Deploy Ghost

```yaml
services:

 ghost:
   image: ghost:6-alpine
   container_name: ghost
   restart: unless-stopped
   depends_on:
     - db
   ports:
     - 8085:2368
   environment:
     database__client: mysql
     database__connection__host: db
     database__connection__user: root
     database__connection__password: your-root-password
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
   image: mysql:8.0
   container_name: ghost_db
   restart: unless-stopped
   environment:
     MYSQL_ROOT_PASSWORD: your-root-password
   volumes:
     - /mnt/tank/configs/ghost/db:/var/lib/mysql
```

1. Change the default password (generate one with `openssl rand -hex 24`). `database__connection__password` in the ghost service must match `MYSQL_ROOT_PASSWORD` in the db service.
1. Update the `url` field to match your FQDN. No domain yet? Use the LAN address instead, for example `http://192.168.1.10:8085`, and change it later.

> [!WARNING]
> **Make `db` its own dataset with Case Sensitivity set to Sensitive.** If your `configs` dataset was created with the SMB preset it is case-insensitive, and every dataset or folder you create under it inherits that. MySQL then initializes with `lower_case_table_names=2`, which is non-standard on Linux and can never be changed afterwards. Create `configs/ghost/db` as a dataset with the **Generic** preset (Case Sensitivity: **Sensitive**) before the first start. Case sensitivity can only be chosen when a dataset is created.

> [!NOTE]
> You can deploy as many of these containers as you want for each website you want to host. Be sure to change the external port for each one. 

> [!WARNING]
> Upgrading from an older version of this guide? Back up `/mnt/tank/configs/ghost`, read the Ghost 6 [update guide](https://docs.ghost.org/update/), and either keep your old content mount line or move everything except `db` into `content/`.


# 2 · Logging In

Once you have deployed this container go to https://example.com/ghost to login and create your account.

> [!NOTE]
> The first start takes several minutes. Until MySQL has finished initializing, Ghost exits with `ECONNREFUSED ...:3306` and restarts on its own, then it spends a few minutes creating its tables. That is normal. Watch `docker logs -f ghost` until it says `Ghost booted`.

# 3 · Monitoring With Umami

In order to enable monitoring, go to the **Settings** gear icon in the lower left corner, then select **Code Injection** section (under **Advanced**), and in the **Site Header** section paste the Tracking Code from the website you created in Umami.

# 4 · Getting Mail Working

You don't need mail to work to host content on Ghost. If you want to have subscribers sign up and get emails when you update content however, you will need email set up. You will also need email configured if you want to have multiple contributors with individual accounts on your platform.

Ghost sends two different kinds of email, and they are configured in two different places:

- **Transactional email** (signup confirmations, login links, staff invites, password resets) goes over plain SMTP. Any provider works here: Mailgun, Zoho, Gmail, your own mail server. This is the `mail__` block in the compose file.
- **Newsletters** (a post emailed to all of your subscribers) go through Mailgun's API, set up inside Ghost's admin settings. Mailgun is the only bulk provider Ghost supports, and Mailgun's free plan includes 100 emails per day.

> [!NOTE]
> Plain SMTP providers like Zoho or Gmail can only do the transactional half. Their terms do not allow bulk sending, and Ghost will not send newsletters over SMTP anyway. If you want newsletters without Mailgun, community projects such as [mailgun-ses-proxy](https://github.com/jcastro/mailgun-ses-proxy) pretend to be Mailgun and send through Amazon SES instead. They are unofficial and can break on a Ghost update.

## 4.1 Mailgun

Navigate to [mailgun](https://signup.mailgun.com/new/signup). Enter all the information keeping the default **Foundation** **50k Trial** selected (it's free for the first month, then becomes a paid plan unless you downgrade to the free plan). Once you have an account, login to your account on mailgun and follow the steps in [this video](https://youtu.be/YnjYWhceepU?feature=shared&t=227) on the correct way to get email configured.

## 4.2 Transactional Email

Once you get to the 10 min mark in the video, he switches gears from bulk email to transactional email. The video edits `config.production.json` inside the container. **Do not do that with Docker:** that file is part of the image, not your mounted `content` folder, so the next image update or container recreate silently throws your mail settings away. Put the SMTP settings in the compose file instead:

::block-steps
1.  **Get your SMTP credentials:** In Mailgun, open your sending domain and create SMTP credentials. Note the login (for example `postmaster@mg.example.com`) and the password.
2.  **Fill in the `mail__` lines in your compose file:**

    ```yaml
         mail__transport: SMTP
         mail__options__service: Mailgun
         mail__options__host: smtp.mailgun.org
         mail__options__port: 587
         mail__options__secure: false
         mail__options__auth__user: postmaster@mg.example.com
         mail__options__auth__pass: your-smtp-password
         mail__from: noreply@example.com
    ```

    Using another SMTP provider? Swap in its host, port and login. Port 465 needs `mail__options__secure: true`.
3.  **Recreate the container:** Run `docker compose up -d` (a plain restart does not pick up new environment variables), and you're done!
::

# 5 · Migrating From Wordpress

> [!NOTE]
> In the event you want to move all your content from Wordpress, there are a few steps to take. Official instructions can be found [here](https://ghost.org/docs/migration/wordpress/). 


## 5.1 Posts and Pages

Install the [Ghost Export Plugin](https://wordpress.org/plugins/ghost/) to your WP instance. I could not get the full .zip with all the images and such to work, but if you can, all you need to do is export your site as a zip file and upload it to Ghost (Ghost > Settings > Import/Export > Universal Import). For those who can't get the full .zip working, use the JSON instead, knowing it won't contain any of your pictures/media.

## 5.2 Redirects

If you left the WP settings for the posts default, it uses a directory tree which includes the /year/month/date/post-name format. Ghost does not use that, so you need to set up 301 redirects in Ghost. Create a .yaml file with these contents *keeping the spacing exactly as you see it*:

```yaml
301:
  ^\/(\d{4})\/(\d{2})\/(\d{2})\/(.*)$: /$4
```

After that, upload this in the Labs section (Ghost > Settings > Labs > Redirects).

