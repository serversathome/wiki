---
title: Pasteguard
description: A guide to deploying Pasteguard
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-02-04T12:53:46.440Z
---

# What is PasteGuard?

**PasteGuard** is a privacy proxy for LLMs that automatically masks personal data (PII) and secrets before sending prompts to AI providers like OpenAI or Anthropic. It detects names, locations, contact details, financial identifiers and secrets in multiple languages, ensuring your private information never leaves your infrastructure. PasteGuard exposes OpenAI-compatible, Anthropic and Codex endpoints, and its detector combines checksums and format checks with a GLiNER-based semantic backend.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PasteGuard

## 1.1 Create the Configuration

PasteGuard now ships a prebuilt all-in-one image (proxy and PII detector in one container).

1. Create the stack folder and download the example config in your TrueNAS shell:
    ```bash
    mkdir -p /mnt/tank/stacks/pasteguard/data
    cd /mnt/tank/stacks/pasteguard
    curl -fsSL -o config.yaml https://raw.githubusercontent.com/sgasser/pasteguard/main/config.example.yaml
    ```

1. Edit `config.yaml` to configure your providers and detection settings. See the Configuration section below for details.

## 1.2 Deploy the Stack

```yaml
services:
  pasteguard:
    image: ghcr.io/sgasser/pasteguard:latest
    container_name: pasteguard
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/stacks/pasteguard/config.yaml:/pasteguard/config.yaml:ro
      - /mnt/tank/stacks/pasteguard/data:/pasteguard/data
    restart: unless-stopped
```

> 
> Point your applications to `http://your-server:3000/openai/v1` instead of `https://api.openai.com/v1` to route requests through PasteGuard. Anthropic clients use `http://your-server:3000/anthropic` and Codex CLI uses `http://your-server:3000/codex`.
{.is-success}

# 2 · Configuration

## 2.1 Operating Modes

PasteGuard operates in two privacy modes:

| Mode | Description |
|------|-------------|
| **Mask** (default) | Replaces PII with placeholders before sending to upstream LLM, then unmasks the response automatically |
| **Route** | Sends requests containing PII to a local LLM (Ollama, vLLM, LocalAI), while clean requests go to your upstream provider |

**Mask Mode Example:**
```
You send:           "Email john@acme.com about the meeting with Sarah Miller"
LLM receives:       "Email <EMAIL_1> about the meeting with <PERSON_1>"
LLM responds:       "I'll contact <EMAIL_1> to schedule with <PERSON_1>..."
You receive:        "I'll contact john@acme.com to schedule with Sarah Miller..."
```

## 2.2 Mask Mode Configuration

```yaml
mode: mask
providers:
  openai:
    base_url: https://api.openai.com/v1
  anthropic:          # optional, enables /anthropic
    base_url: https://api.anthropic.com
masking:
  show_markers: false
  marker_text: "[protected]"
  # allowlist:        # text that is never masked
  #   - pattern: "Company Name Inc."
  # denylist:         # text that is always masked
  #   - pattern: "ProjectX"
  #     type: PROJECT_NAME
```

## 2.3 Route Mode Configuration

```yaml
mode: route
providers:
  openai:
    base_url: https://api.openai.com/v1
local:
  type: ollama        # or openai for vLLM, LocalAI, LM Studio
  base_url: http://<ollama-ip>:11434
  model: llama3.2
```

> 
> Route mode requires a local LLM such as Ollama, vLLM, or LocalAI to be running and accessible.
{.is-warning}

# 3 · PII Detection

## 3.1 Detected Entity Types

PasteGuard detects the following PII types by default:

| Type | Examples |
|------|----------|
| Names | John Smith, Sarah Miller |
| Emails | john@acme.com |
| Phone Numbers | +1 555 123 4567 |
| Credit Cards | 4111-1111-1111-1111 |
| IBANs | DE89 3704 0044 0532 0130 00 |
| IP Addresses | 192.168.1.1 |
| Locations | New York, Berlin |
| EU VAT Numbers | DE123456789 |
{.dense}

## 3.2 PII Configuration

```yaml
pii_detection:
  detector_url: ${DETECTOR_URL:-http://localhost:5002}  # built into the all-in-one image
  detector_timeout: ${DETECTOR_TIMEOUT:-30}
  phone_regions: []   # e.g. [US, GB, DE] for national-format numbers
  score_threshold: 0.7
  entities:
    - PERSON
    - LOCATION
    - EMAIL_ADDRESS
    - PHONE_NUMBER
    - CREDIT_CARD
    - IBAN_CODE
    - IP_ADDRESS
    - VAT_CODE
```

# 4 · Secrets Detection (Secrets Shield)

PasteGuard includes a Secrets Shield that detects credentials and API keys before they reach any LLM.

## 4.1 Detected Secret Types

| Type | Pattern |
|------|---------|
| OpenSSH Private Keys | `-----BEGIN OPENSSH PRIVATE KEY-----` |
| PEM Private Keys | `-----BEGIN RSA PRIVATE KEY-----` |
| `sk-` API Keys (OpenAI, Anthropic, Stripe, ...) | `sk-proj-...`, `sk-...` |
| AWS Access Keys | `AKIA...` (20 chars) |
| GitHub Tokens | `ghp_...`, `gho_...`, `ghu_...` |
| JWT Tokens | `eyJ...` (three base64 segments) |
| Bearer Tokens | `Bearer ...` (20+ char tokens) |
| Passwords / Secrets in env files | `PASSWORD=...`, `SECRET=...` |
| Connection Strings | `postgres://user:pass@host/db` |
{.dense}

## 4.2 Secrets Configuration

```yaml
secrets_detection:
  enabled: true
  action: mask  # mask | block | route_local
  entities:
    - OPENSSH_PRIVATE_KEY
    - PEM_PRIVATE_KEY
    - API_KEY_SK
    - API_KEY_AWS
    - API_KEY_GITHUB
    - JWT_TOKEN
    - BEARER_TOKEN
    - ENV_PASSWORD
    - ENV_SECRET
    - CONNECTION_STRING
  max_scan_chars: 200000
  log_detected_types: true
```

| Action | Behavior |
|--------|----------|
| **mask** | Replaces secrets with placeholders, unmasks in response |
| **block** | Returns HTTP 400 error, request never reaches LLM |
| **route_local** | Routes to local LLM when secrets detected (route mode only) |
{.dense}

> 
> The default action is `mask`. Set `action: block` if you would rather reject any request containing a secret. Detected secrets are never logged in their original form.
{.is-danger}

# 5 · Dashboard & Authentication

PasteGuard includes a built-in monitoring dashboard at the root URL, `http://your-server:3000`. Request logs are stored in SQLite (`./data/pasteguard.db`) by default, or Postgres with `logging.driver: postgres`, and kept for `logging.retention_days` (30 by default). To secure the dashboard, configure authentication in `config.yaml`:

```yaml
dashboard:
  auth:
    username: admin
    password: ${DASHBOARD_PASSWORD}
```

Set the `DASHBOARD_PASSWORD` environment variable or replace `${DASHBOARD_PASSWORD}` with your desired password.


# 6 · Language Support

PII detection is multilingual. The detector uses checksums and format checks for structured values (cards, IBANs, VAT numbers) and a GLiNER semantic backend for names and places, so there is no per-language build step any more. Use `phone_regions` in `config.yaml` if you need national-format phone numbers detected; numbers in `+` international format work everywhere. See the [PasteGuard docs](https://pasteguard.com/docs) for custom GLiNER models.



# <img src="/youtube.png" class="tab-icon"> 7 · Video

*Video coming soon*