---
title: Wiki.js
description: A guide to installing Wiki.js in docker via compose
published: true
date: '2026-11-07T12:57:22.849Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:09:35.215Z'
---

# What is Wiki.js?

**Wiki.js** is an open source wiki software that works on any platform and database. It offers features such as authentication, media, themes, integrations and more.

# :logos:docker-icon: 1 · Deploy Wiki.js

```yaml
services:
  db:
    image: postgres:18
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/wikijs:/var/lib/postgresql

  wiki:
    image: ghcr.io/requarks/wiki:3
    depends_on:
      - db
    environment:
      ADMIN_EMAIL: user@example.com
      ADMIN_PASS: SuperSecret123
      DB_HOST: db
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "3000:3000"

```

1. Change all the default user names and passwords. Make sure when you change them in the wiki service you mirror those changes in the db service. 


# 2 · Making it Beautiful

The strength of any editor is the Markdown function. The easiest way to start with Wiki.js is with the Visual Editor, which operates like Microsoft Word. However, all of the most powerful functions are only availble using the markdown editor. 

## 2.1 Converting Editors

To change a page from Visual Editor to the Markdown editor:

::block-steps
1. Navigate to the Page Actions button while viewing the page you would like to convert
1. Click **Convert**
1. Select **Markdown** and hit **Convert**
::

> [!WARNING]
> A note when you do this, if you convert back to Visual Editor all the formatting will be lost from Markdown

> [!TIP]
> All the markdown instructions can be found [here](https://docs.requarks.io/en/editors/markdown)

## 2.2 Navigation Icons

By default, Wiki.js uses the Material Designs icon set for naviagtion. The best way to do navigation in my opinion is to go to **Administration Settings** ⚙️ → **Navigation** ➤ and set the **Navigation Mode** to **Custom Naviagtion** and change the default *mdi-chevron-right* to something from [here](https://pictogrammers.com/library/mdi/).


# 3 · Syncing With GitHub

[GitHub](https://www.github.com) is the most popular git source control provider.

## 3.1 Generate a new key

::block-steps
1. Open Terminal.
2. Enter the command:
   ```bash
   ssh-keygen -t rsa -b 4096
   ```
3. When prompted to save the generated file, enter a path which can be accessed by Wiki.js *(e.g. `/etc/wiki/github.pem`)* and press **Enter**.
4. Leave the passphrase empty and press **Enter** twice. Password-protected keys will NOT work.
::

> [!NOTE]
> On Windows, you can use [Git Bash](https://git-scm.com/download/win) or Windows Subsystem for Linux (WSL) distributions like [Ubuntu for Windows](https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6) to run the commands above. You can also generate keys manually using the [puttygen](https://www.ssh.com/ssh/putty/download) utility.

## 3.2 Add the key to GitHub

::block-steps
1. Create a new GitHub repository.
2. Click on the **Settings** tab.
3. Click on the **Deploy Keys** in the left navigation menu.
4. Click the **Add deploy key** button.
5. Enter a name of your choice for this key (e.g. wiki) and paste the contents of the public key generated earlier. *(file ending in `.pub`)*
6. Make sure the **Allow write access** is checked.
7. Click the **Add key** button.
::

## 3.3 Configure Wiki.js

::block-steps
1. In the Administration Area, click on **Storage** in the left navigation menu.
2. Make sure the **Git** storage target is checked.
3. Click on the **Git** tab.
4. Enter the following settings:
   - Authentication Type: `ssh`
   - Repository URI: *On your GitHub repository page, in the **Code** tab, click on the **Clone or download** green button and copy the URI shown below **Clone with SSH**.*
   - Branch: `main`
   - SSH Private Key Path: *The path to your private key generated earlier.*
   - Username: *Empty*
   - Password: *Empty*
   - Default Author Email: *Should match your GitHub account email.*
   - Default Author Name: *Should match your GitHub account name.*
   - Local Repository Path: *Choose where the repo will be cloned locally or leave the default `./data/repo` value.*
   - Verify SSL Certificate: **On**
5. Set the **Sync Direction** to **Bi-directional**.
6. Set the **Sync Schedule** to **5 minutes**.
7. Click the **Apply Changes** button at the top of the page.
8. Wait for the **Status** panel to update. A new entry for **Git** should appear in green. If the bar is red, it means you have an error in your configuration. Go back to the Git tab, fix the error and try again.
::

# :logos:youtube-icon: 4 · Video
::block-youtube{url="https://youtu.be/hfhP5WBAXmg"}
::
