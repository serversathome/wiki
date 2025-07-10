---
title: Recommendarr
description: A guide to deploying Recommendarr via docker compose
published: true
date: 2025-07-10T18:34:27.876Z
tags: 
editor: markdown
dateCreated: 2025-05-15T10:52:38.357Z
---

# ![](/recommendarr.png){class="tab-icon"} What is Recommendarr?
Recommendarr is a web application that generates personalized TV show and movie recommendations based on your Sonarr, Radarr, Plex, and Jellyfin libraries using AI.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Recommendarr

```yaml
services:
  recommendarr:
    image: tannermiddleton/recommendarr:latest
    container_name: recommendarr
    ports:
      - 3006:3000
    environment:
      - NODE_ENV=production
      - DOCKER_ENV=false
      - PORT=3000
      - PUBLIC_URL=http://your-server-IP:3006
    volumes:
      - /mnt/tank/configs/recommendarr:/app/server/data
    restart: unless-stopped
```
 
> Be sure to change the `PUBLIC_URL` to your server address
{.is-warning}

# 2 · Logging In
1. Navigate to `http://IP:3006` to visit the login page
1. The default username is `admin`
1. The default password is `1234`

# 3 · Recommendarr Configuration
1. Navigate to **Settings → Sonarr**
1. Enter the **Sonarr URL**
1. Enter the **Sonarr API Key**
1. Do the same for **Radarr**
1. Navigate to **AI Service** and add an **API URL** and **API KEY**

> Optionally add a TMDB API Key as well as Plex or Jellyfin
{.is-info}

# 4 · Using AI
Recommendar gives you the option of using your existing API Key from ChatGPT, a locally hosted AI model, or an API Key from [OpenRouterAI](https://openrouter.ai/). However, any self-hosted Large Language Model (LLM) that exposes an API conforming to the OpenAI chat completions standard can be used.

## 4.1 Getting an OpenRouter API Key
1. Navigate to [OpenRouterAI](https://openrouter.ai/)
1. Create a login
1. Navigate to the **☰ Menu** in the top right corner **Settings → API Keys**
1. Click the **Create API Key** button and copy the key shown

> The URL for OpenRouterAI is `https://openrouter.ai/api/v1`
{.is-info}

# 5 · Getting Recommendations
1. Navigate to the **Recommendations** tab of Recommendarr
1. Select your model
1. Select **Content Type** (either TV Shows or Movies)
1. Select your **Content Language**
1. Click the big blue button at the bottom to get recommendations

# 6 · Video
[![](/2025-05-15-free-ai-media-recommendations-wi-promo-card.png)](https://www.patreon.com/posts/free-ai-media-129054870)