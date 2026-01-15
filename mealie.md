---
title: Mealie
description: A guide to deploying Mealie
published: true
date: 2026-01-15T15:30:07.828Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:17.048Z
---

# ![](/mealie.png){class="tab-icon"} What is Mealie?
Mealie is a self hosted recipe manager and meal planner with a RestAPI backend and a reactive frontend application built in Vue for a pleasant user experience for the whole family. Easily add recipes into your database by providing the url and Mealie will automatically import the relevant data or add a family recipe with the UI editor. Mealie also provides an API for interactions from 3rd party applications.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Mealie
```yaml
services:
  mealie:
    image: ghcr.io/mealie-recipes/mealie:latest
    container_name: mealie
    restart: unless-stopped
    ports:
      - 9925:9000
    volumes:
      - /mnt/tank/configs/mealie:/app/data/
    environment:
      ALLOW_SIGNUP: "false"
      PUID: 1000
      PGID: 1000
      TZ: America/New_York
      MAX_WORKERS: 1
      WEB_CONCURRENCY: 1
      BASE_URL: 
      OPENAI_API_KEY: 
      OPENAI_ENABLE_IMAGE_SERVICES: "false"
      OPENAI_SEND_DATABASE_DATA: "false"
```

> See more environment variable options and explanations [here](https://docs.mealie.io/documentation/getting-started/installation/backend-config/)
{.is-info}


# 2 · Why Add AI?
- The OpenAI Ingredient Parser can be used as an alternative to the NLP and Brute Force parsers. Simply choose the OpenAI parser while parsing ingredients 
- When importing a recipe via URL, if the default recipe scraper is unable to read the recipe data from a webpage, the webpage contents will be parsed by OpenAI 
- You can import an image of a written recipe, which is sent to OpenAI and imported into Mealie. The recipe can be hand-written or typed, as long as the text is in the picture. You can also optionally have OpenAI translate the recipe into your own language 

If you have another service you'd like to use in-place of OpenAI, you can configure Mealie to use that instead, as long as it has an OpenAI-compatible API. For instance, a common self-hosted alternative to OpenAI is Ollama. To use Ollama with Mealie, change your `OPENAI_BASE_URL` to http://localhost:11434/v1 (where http://localhost:11434 is wherever you're hosting Ollama, and /v1 enables the OpenAI-compatible endpoints). Note that you must provide an API key, even though it is ultimately ignored by Ollama.

# <img src="/docker.png" class="tab-icon"> 3 · Grab Recipes from Social
Have you found a recipe on social media and don’t want to write it out yourself? This tool lets you import recipes from videos directly into Mealie.

> See the [GitHub repo](https://github.com/GerardPolloRebozado/social-to-mealie) for more information!
{.is-info}


```yaml
services:
  social-to-mealie:
    restart: unless-stopped
    image: ghcr.io/gerardpollorebozado/social-to-mealie:latest
    container_name: social-to-mealie
    environment:
      - OPENAI_URL=https://api.openai.com/v1 #URL of api endpoint of AI provider
      - OPENAI_API_KEY=
      - WHISPER_MODEL=whisper-1 # this model will be used to transcribe the audio to text
      - MEALIE_URL=https://mealie.example.com
      - MEALIE_API_KEY=
provided in Spanish
    ports:
      - 4000:3000
    security_opt:
      - no-new-privileges:true
```