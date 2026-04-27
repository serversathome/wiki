---
title: Zublo
description: A guide to dpeloying Zublo
published: true
date: 2026-04-27T17:44:21.112Z
tags: 
editor: markdown
dateCreated: 2026-04-27T17:44:21.112Z
---

# <img src="/zublo.png" class="tab-icon"> What is Zublo?

**Zublo** is an open-source, self-hostable personal subscription tracker built for homelabbers and privacy-minded users who want every recurring payment in one place. It ships as a single Docker container with a React frontend and a PocketBase backend (SQLite-backed), keeping the deployment story dead simple — one container, one volume, one port.

Beyond just tracking subscriptions, Zublo includes a real AI layer with chat-based workflows, spending recommendations, and pluggable LLM providers (Google Gemini, OpenAI, Ollama, and any OpenAI-compatible gateway like OpenRouter, Groq, or Mistral). It also offers a calendar view of upcoming payments, multi-currency support with exchange-rate sync, REST API access via scoped keys, and TOTP-based 2FA — all without becoming a bloated full-finance suite.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Zublo

```yaml
services:
  zublo:
    image: ghcr.io/danielalves96/zublo:latest
    container_name: zublo
    restart: unless-stopped
    ports:
      - "9597:9597"
    environment:
      PB_ENCRYPTION_KEY: "change-me-in-production"
    volumes:
      - /mnt/tank/configs/zublo:/pb/pb_data
```


1. The PocketBase admin panel is available at `http://<your-host>:9597/_/`.
1. The REST API is exposed at `http://<your-host>:9597/api/`.

 
> Once you have the first admin account created, **immediately disable open registration** in the PocketBase admin panel (`/_/`) under **Settings → Auth providers** to prevent random users from signing up if you expose Zublo to the internet.
{.is-danger}

# 2 · Configuration

## 2.1 Initial Setup

After registering your first account, you'll be dropped into the dashboard. Before adding subscriptions, take a minute to set up:

1. **Default currency** — set this in **Settings** so new subscriptions inherit it
2. **Notification preferences** — configure when and how you want reminders for upcoming payments
3. **Two-factor authentication** — enable TOTP under your user profile for an extra layer of protection
4. **Categories** — create categories that match how you mentally bucket your subscriptions (Streaming, Software, Cloud, Domains, etc.)

## 2.2 Adding Subscriptions

For each recurring payment, Zublo tracks the billing cycle, due date, amount, currency, and category. You can also attach notes for context — like the email address tied to the account or which credit card it bills against.

The calendar view will then surface upcoming payments at a glance, and the statistics view breaks down spend by category, currency, and trend over time.

## 2.3 AI Provider Setup

Zublo's AI layer is provider-agnostic. Head to **Settings → AI** to wire it up:

- **Google Gemini** — drop in your API key from Google AI Studio
- **OpenAI** — paste a standard `sk-...` key
- **Ollama** — point it at your local Ollama instance (e.g., `http://ollama:11434`) for fully self-hosted inference
- **OpenAI-compatible** — works with OpenRouter, Groq, Mistral, or any gateway exposing the OpenAI chat completions API

Once configured, the chat interface can answer questions about your spending, suggest subscriptions to review, and surface patterns you might miss.

> 
> If you're already running Ollama in your homelab, point Zublo at it and you've got a fully self-hosted, AI-enabled subscription tracker with zero data leaving your network.
{.is-success}





