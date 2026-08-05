---
title: LinkBreeze
description: A guide to deploying LinkBreeze
published: true
date: 2026-08-05T20:25:52.869Z
tags: 
editor: markdown
dateCreated: 2026-08-05T20:25:52.869Z
---

# What is LinkBreeze?

**LinkBreeze** is a self-hosted link-in-bio platform — the open-source alternative to Linktree. It gives you a single public page that holds all of your links, plus a fast admin dashboard for managing them, built-in privacy-first analytics, auto-generated QR codes, and a full theme engine.

It runs as a single container backed by SQLite (no external database), the public page is 100% server-rendered with zero client JavaScript, and the whole thing is MIT licensed. If you're a creator who's tired of paying a monthly subscription just to host a list of links, this is the one to self-host.




# 1 · Deploy LinkBreeze

```yaml
services:
  linkbreeze:
    image: ghcr.io/manak-hash/linkbreeze:latest
    container_name: linkbreeze
    environment:
      - DATABASE_PATH=/app/data/linkbreeze.db
      - SECRET_KEY=CHANGE_THIS_TO_A_LONG_RANDOM_STRING
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/linkbreeze:/app/data
```

 
> Generate a strong signing key with `openssl rand -hex 32` and paste the output into `SECRET_KEY`. This key signs your admin session cookies — if you leave it at a default or change it later, every existing session is invalidated and you'll be logged out.
{.is-warning}


# 2 · First-Time Setup

## 2.1 Setup Wizard

On first launch, LinkBreeze walks you through creating your admin account and your public page. The whole thing takes under a minute.

1. Open `http://<your-server-ip>:3000`
2. Create your **admin username and password**
3. Set your **page slug** — this is the public URL, e.g. `/johndoe`
4. Set your **display name**, **bio**, and **avatar**

Your public page now lives at `http://<your-server-ip>:3000/<your-slug>`, and the admin dashboard is at `/admin`.

> 
> Use a real password here. If you expose this page to the internet, the admin panel is exposed with it.
{.is-danger}

## 2.3 Adding Links

1. Log into **/admin** and open the **Links** tab
2. Click **Add Link**, then set the title, URL, and optional thumbnail
3. Drag the handles to reorder — the public page updates immediately
4. Open a link's options to set a **schedule** (start/end date and time) or turn it into an **embed widget**

# 3 · Theming

LinkBreeze ships with 9 presets: **Aurora** (the animated flagship), **Glassmorphism**, **Neon Cyberpunk**, **Editorial Paper**, **Terminal Mono**, **Pastel Soft**, **Brutalist**, **Retro Sunset**, and **Minimal Light**.

Under the hood the theme engine is a CSS custom property token system (`--lb-*`), so the customizer can change nearly everything without touching code:

| Area | What you can change |
|------|---------------------|
| Background | 8 types (solid, gradient, radial, mesh, aurora, animated, image, pattern) plus angle, overlay, opacity |
| Colors | Accent, secondary, text, muted text, card background, card border |
| Typography | 10 Google Fonts, font scale, weight, letter spacing |
| Card Style | 6 link styles (pill, rounded, sharp, glass, outline, neon), hover effects, size, radius, border, shadow |
| Layout | Container width, alignment, density |
| Effects | Glow, glass blur, noise texture, reveal animation |


Themes can be duplicated, exported, and imported — handy if you want to keep a working copy before experimenting, or share a look between instances.

> 
> Need something the customizer can't do? The **Custom CSS** box in the theme settings injects raw CSS directly into the public page.
{.is-info}

