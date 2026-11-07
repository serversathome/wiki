---
title: Wiki.js
description: A guide to installing Wiki.js 3 in docker via compose
published: true
date: '2026-11-07T19:37:23.412Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:09:35.215Z'
---

# What is Wiki.js?

**Wiki.js** is an open source wiki that runs on PostgreSQL. Version 3 is a full rewrite: a new Vue 3 interface, a Visual and a Markdown editor that share the same source, content blocks (tabs, YouTube, diagrams, link grids and more), live collaborative editing, per-folder navigation menus, and storage targets such as Git, S3 and local disk.

> [!NOTE]
> This guide covers Wiki.js **3.x**. If you are still running 2.x, see **2 · Upgrading from 2.x** below before you touch anything: a 3.x container cannot be pointed at a 2.x database.

# :logos:docker-icon: 1 · Deploy Wiki.js

```yaml
services:
  db:
    image: postgres:18
    user: "568:568"
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



> [!WARNING] PostgreSQL 17 or newer is required
> Wiki.js 3 only runs on PostgreSQL 17+, and MySQL, MariaDB, SQLite and MSSQL are no longer supported. Starting with `postgres:18`, the volume is mounted at `/var/lib/postgresql` (not `/var/lib/postgresql/data`), so do not reuse an old 2.x database folder or mount path.

> [!NOTE]
> - The image tag is `:3`, which tracks the latest 3.x release. Wiki.js 3 does not publish a `latest` tag, so `:latest` will not pull. Pin a version such as `:3.1` if you prefer.
> - The wiki container needs no volume: pages, files, settings and users all live in the database. Leave `user:` off that service: it runs as its own built-in non-root user, and its working folder inside the image is only writable by that user.

# 2 · Upgrading from 2.x

There is no in-place upgrade from 2.x. You export from the old wiki and import into a fresh 3.x install:
::block-steps
1. Update the 2.x wiki to **2.5.317**, the last 2.x release.
1. In the 2.x **Administration** area, open **Utilities** and run **Export for Wiki.js 3.x**. Keep every box checked and download the `.wkbackup` file.
1. Deploy a new 3.x stack using the compose above, with a new database folder.
1. Log in as the admin, open **Administration → Utilities** and click **Proceed** next to **Import from Wiki.js 2.x**.
1. Keep the default options, select the `.wkbackup` file and start the import. Watch the progress for errors.
::

> [!WARNING] Not everything comes across
> Storage targets (including Git), authentication, comments, analytics, security settings, the site logo and the hostname are not imported. Set those up again by hand. The import can be run more than once; existing items are skipped unless you turn on overwrite.

After the import, check pages for 2.x markdown that 3.x renders differently:

- `# Title {.tabset}` tabs show up as plain text. Rewrite them with the `:::block-tabs` content block.
- Mermaid, PlantUML, Kroki and KaTeX diagrams only render inside their content block (`::block-diagram` for Mermaid).
- Callouts written as `> text` followed by `{.is-info}` still work, but the new style is GitHub alerts: `> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`, `> [!CAUTION]` and `> [!IMPORTANT]`.
- Relative links now resolve from the page's parent folder.
- The old YouTube code injection from 2.x is no longer needed, since `::block-youtube` is built in. Remove it from **Theme → Code Injection**.

# 3 · Editors

## 3.1 Visual vs Markdown

The **Visual Editor** works like a word processor and the **Markdown Editor** shows the source with a live preview beside it. In 3.x both write the same Markdown source, so you can move a page between them without losing formatting. Content blocks such as tabs and link grids are easiest to write in the Markdown editor.

Editors can be turned on and off under **Administration → Editors**.

## 3.2 Converting a page

::block-steps
1. Open the page you want to convert
1. Open the **Page Actions** menu and click **Convert Page**
1. Pick the editor the page should open with from now on and click **Convert**
::

> [!NOTE]
> Wiki.js previews the page before and after. If converting would change how the page renders, the conversion is refused instead of damaging the page, so nothing is lost.

> [!TIP]
> All the markdown syntax can be found [here](https://docs.js.wiki/guide/markdown) and the content blocks [here](https://docs.js.wiki/guide/blocks).

# 4 · Navigation

Navigation menus in 3.x can be different for each folder, and the icons come from [Iconify](https://icon-sets.iconify.design/) instead of only Material Design Icons.
::block-steps
1. Go to **Administration → Icons**, click **Add Icon Set** and add the sets you want (for example `mdi`, `logos` and `simple-icons`). Icons from enabled sets can be used in menus, page icons and content blocks.
1. On any page, click **Edit Nav** at the bottom of the sidebar.
1. Choose a mode for that path: **Inherit**, **Override Current + Descendants**, **Override Current Only**, **Hide Current + Descendants** or **Hide Current Only**.
1. Click **Edit Menu Items**, then **Add** a Header, Link or Separator. Links and Menus have an **Icon** field that opens the icon picker.
1. Click **Save**.
::

> [!TIP]
> **Administration → Navigation** lists every menu on the site and has automations that add new pages to their parent folder's menu, move links when a page moves, and clean up menus nothing uses.

# 5 · Syncing With GitHub

[GitHub](https://www.github.com) is the most popular git source control provider. With the Git storage target, every page save becomes a commit, and changes pushed to the repository show up in the wiki on the next sync.

## 5.1 Generate a new key

::block-steps
1. Open a terminal
1. Enter the command:
   ```bash
   ssh-keygen -t ed25519 -C "wikijs" -f ./wikijs-github
   ```
1. Leave the passphrase empty and press **Enter** twice. Password-protected keys will NOT work.
1. This makes two files: `wikijs-github` (the private key, for Wiki.js) and `wikijs-github.pub` (the public key, for GitHub)
::

> [!NOTE]
> On Windows, you can run the same command in PowerShell or [Git Bash](https://git-scm.com/download/win).

## 5.2 Add the key to GitHub

::block-steps
1. Create a new GitHub repository
1. Click on the **Settings** tab
1. Click **Deploy keys** in the left navigation menu
1. Click the **Add deploy key** button
1. Enter a name of your choice for this key (e.g. wiki) and paste the contents of the public key *(the file ending in `.pub`)*
1. Make sure **Allow write access** is checked
1. Click the **Add key** button
::

## 5.3 Configure Wiki.js

::block-steps
1. In the **Administration** area, click **Storage** in the left navigation menu
1. Click **Git** and turn it on
1. Enter the following settings:
   - Authentication Type: **SSH**
   - Repository URI: *On your GitHub repository page, click the green **Code** button, choose **SSH** and copy the URI (e.g. `git@github.com:you/wiki.git`)*
   - Branch: `main`
   - Sync Direction: **Sync** *(pulls, then pushes)*
   - SSH Private Key Mode: **Inline Contents**
   - SSH Private Key Contents: *Paste the contents of the private key file (the one without `.pub`)*
   - Verify SSL Certificate: **On**
   - Default Author Name / Email: *Used for commits no single user made, such as scheduled syncs*
   - Local Repository Path: *Leave the default `./data/repo`*
1. Open the **Configuration** tab and set **Sync Interval** to `5m`
1. Click **Save**
1. Open the Git target's **Actions** and click **Add Untracked Changes** to commit the pages that already exist in the wiki
1. Check the **Status** card. Green means it is working; a red status shows the error from the last sync
::

> [!NOTE]
> - The sync interval now lives on the **Configuration** tab and applies to every storage target on the site. **Force Sync** in the target's Actions runs one immediately.
> - The other Sync Directions are **Push only** (the wiki always wins and force-pushes) and **Pull only** (the repository is the authority and the wiki never pushes).
> - Turn off **Add Locale Prefix** on the **Configuration** tab if you want pages at the root of the repo instead of under an `en/` folder.
> - Connecting to a repository that already has content? The first sync does not take it in. Run **Import Everything** from the Actions afterwards.

> [!CAUTION] The repository can delete pages
> A pull is authoritative. Deleting or renaming a `.md` file in the repo deletes or moves the page in the wiki, so anyone with push access to the repository effectively has write access to your wiki.

# :logos:youtube-icon: 6 · Video

> [!NOTE]
> This video was recorded on Wiki.js 2.x. The deployment and settings above are current for 3.x.

::block-youtube{url="https://youtu.be/hfhP5WBAXmg"}
::
