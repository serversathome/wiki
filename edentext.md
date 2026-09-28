---
title: EdenText
description: A guide to deploying EdenText
published: true
date: 2026-09-28T18:03:54.644Z
tags: 
editor: markdown
dateCreated: 2026-09-28T18:03:54.644Z
---

# What is EdenText?

**EdenText** is a full word processor that runs entirely in your browser. It opens and saves `.odt` and `.docx`, exports PDF, and handles real page layout: headers and footers, footnotes, columns, styles, tables with formulas, tables of contents, citations, tracked changes and threaded comments. There is no backend and no account. Documents never leave the browser, so self-hosting it just means serving the static app from your own server.

<div class="glance">
  <div><span>Port</span><b><code>8096</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/stffnb/edentext"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy EdenText

EdenText stores nothing on the server, so there is no config folder to create.

```yaml
services:
  edentext:
    image: ghcr.io/stffnb/edentext:latest
    container_name: edentext
    user: "568:568"
    tmpfs:
      - /var/cache/nginx
      - /run
    ports:
      - "8096:80"
    restart: unless-stopped
```


> The image is plain Nginx serving static files. Nginx normally starts as root so it can write its cache and PID file. The two `tmpfs` mounts give it writable scratch space, which is what lets it run as the TrueNAS apps user.
{.is-info}

> EdenText is in **beta**. The project runs LibreOffice round-trip checks on every commit, but keep backups of any document you care about until it matures.
{.is-warning}

# 2 · Configuration

## 2.1 Where Your Documents Live

Everything happens in the browser. Documents, settings and recovery copies are kept in that browser's local storage, and the last three versions of the open document are kept in case one fails to load. That means:

- Clearing site data for your EdenText URL clears your in-browser documents
- A document opened on your laptop is not visible from your phone
- Your server never sees the content, so there is nothing to back up on the server side

Save anything important as `.odt` or `.docx` to your NAS or cloud drive.

## 2.2 Saving Files

In Chrome and Edge, **Save** writes straight back to the file you opened. Firefox and Safari hand each save over as a download instead. Turn on "Always ask where to save" in those browsers so you can choose the location each time.

## 2.3 Install as an App

EdenText is an installable web app and works offline once loaded. Use your browser's **Install app** option on the address bar to give it its own window. Note that browsers only offer install and offline mode over HTTPS, so put it behind a Cloudflare Tunnel or reverse proxy if you want that feature from anything other than `localhost`.

