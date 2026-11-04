---
title: Mealie
description: A guide to deploying Mealie
published: true
date: '2026-11-04T23:31:46.445Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:06:17.048Z'
---

# What is Mealie?
Mealie is a self hosted recipe manager and meal planner with a RestAPI backend and a reactive frontend application built in Vue for a pleasant user experience for the whole family. Easily add recipes into your database by providing the url and Mealie will automatically import the relevant data or add a family recipe with the UI editor. Mealie also provides an API for interactions from 3rd party applications.

# :logos:docker-icon: 1 · Deploy Mealie
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
      PUID: 568
      PGID: 568
      TZ: America/New_York
      UVICORN_WORKERS: 1
      BASE_URL: 
```

> [!NOTE]
> See more environment variable options and explanations [here](https://docs.mealie.io/documentation/getting-started/installation/backend-config/)


# 2 · Why Add AI?
- The AI Ingredient Parser can be used as an alternative to the NLP and Brute Force parsers. Simply choose the AI parser while parsing ingredients 
- When importing a recipe via URL, if the default recipe scraper is unable to read the recipe data from a webpage, the webpage contents will be parsed by AI 
- The **Import with AI** page (*Create → Import with AI*) builds a recipe from pasted text, photos of a written recipe, or a link. Give it a YouTube or Instagram video link and Mealie transcribes the video, as long as you've set up an audio provider. You can also optionally have the recipe translated into your own language 

AI is no longer configured with environment variables. To set it up:

::block-steps
1. Go to your **Group Settings** and open the AI providers section
1. Add a **default provider** (e.g. OpenAI with `gpt-5` and your API key). AI features stay off until a default provider is set
1. Optionally add an **image provider** (for recipes from photos) and an **audio provider** such as `whisper-1` (for video imports)
::

If you have another service you'd like to use in-place of OpenAI, you can configure Mealie to use that instead, as long as it has an OpenAI-compatible API. For instance, a common self-hosted alternative to OpenAI is Ollama. To use Ollama with Mealie, set the provider's `base_url` to http://localhost:11434/v1 (where http://localhost:11434 is wherever you're hosting Ollama, and /v1 enables the OpenAI-compatible endpoints). Note that you must provide an API key, even though it is ultimately ignored by Ollama.

> [!NOTE]
> See the [AI integration docs](https://docs.mealie.io/documentation/getting-started/installation/ai-providers/) for more details.

# :logos:docker-icon: 3 · Grab Recipes from Social
Have you found a recipe on social media and don’t want to write it out yourself? This tool lets you import recipes from videos directly into Mealie.

> [!NOTE]
> Mealie can now import videos itself through **Import with AI** (see section 2) once an audio provider is configured. social-to-mealie is still handy if you want a separate share target or different models.

> [!NOTE]
> See the [GitHub repo](https://github.com/GerardPolloRebozado/social-to-mealie) for more information!


```yaml
services:
  social-to-mealie:
    restart: unless-stopped
    image: ghcr.io/gerardpollorebozado/social-to-mealie:latest
    container_name: social-to-mealie
    environment:
      - OPENAI_URL=https://api.openai.com/v1 #URL of an OPENAI api compatible provider
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - TRANSCRIPTION_MODEL=whisper-1 # this model will be used to transcribe the audio to text
      - TEXT_MODEL=gpt-5-mini # the model you want to use to create the recipe in json using the transcription result
      - MEALIE_URL=https://mealie.example.com # url of you mealie instance
      - MEALIE_API_KEY=${MEALIE_API_KEY}
      # Optionally specify yt-dlp version to use. Use "latest" or a release tag like "2025.11.01".
      # If provided, the container will try to download yt-dlp at startup
      - YTDLP_VERSION=latest
      # Optional, Mealie group name, defaults to "home"
      - MEALIE_GROUP_NAME=home
      # Optional, addition to the prompt, useful for translation needs remove if not needed
      - EXTRA_PROMPT=
      - COOKIES= #Optional, cookie for yt-dlp requests, useful for protected content
    volumes:
      - ./cookies.txt:/app/cookies.txt # Optional, only needed for some content
    ports:
      - 4000:3000
    security_opt:
      - no-new-privileges:true
```
