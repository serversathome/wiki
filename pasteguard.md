---
title: Pasteguard
description: A guide to deploying Pasteguard
published: true
date: 2026-02-04T12:53:46.440Z
tags: 
editor: markdown
dateCreated: 2026-02-04T12:53:46.440Z
---

# What is PasteGuard?

**PasteGuard** is a privacy proxy for LLMs that automatically masks personal data (PII) and secrets before sending prompts to AI providers like OpenAI or Anthropic. It detects 30+ types of sensitive data across 24 languages, ensuring your private information never leaves your infrastructure. PasteGuard is OpenAI API compatible and powered by Microsoft Presidio.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PasteGuard

## 1.1 Clone the Repository
> PasteGuard does not have a docker compose file and must use docker build
{.is-info}

1. Run the following command in the `stacks` directory of your TrueNAS shell:
    ```bash
    git clone https://github.com/sgasser/pasteguard.git
    ```
1. Create Configuration

    ```bash
    cd pasteguard
    cp config.example.yaml config.yaml
    ```

1. Edit `config.yaml` to configure your providers and detection settings. See the Configuration section below for details.

1. Navigate to your container manager and launch the stack


> 
> Point your applications to `http://your-server:3000/openai/v1` instead of `https://api.openai.com/v1` to route requests through PasteGuard.
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
  upstream:
    type: openai
    base_url: https://api.openai.com/v1
masking:
  placeholder_format: "<{TYPE}_{N}>"
  show_markers: false
```

## 2.3 Route Mode Configuration

```yaml
mode: route
providers:
  upstream:
    type: openai
    base_url: https://api.openai.com/v1
  local:
    type: ollama
    base_url: http://localhost:11434
    model: llama3.2
routing:
  default: upstream
  on_pii_detected: local
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
{.dense}

Additional entity types can be enabled: `US_SSN`, `US_PASSPORT`, `CRYPTO`, `NRP`, `MEDICAL_LICENSE`, `URL`.

## 3.2 PII Configuration

```yaml
pii_detection:
  score_threshold: 0.7
  languages:
    - en
    - de
  entities:
    - PERSON
    - EMAIL_ADDRESS
    - PHONE_NUMBER
    - CREDIT_CARD
    - IBAN_CODE
```

# 4 · Secrets Detection (Secrets Shield)

PasteGuard includes a Secrets Shield that detects credentials and API keys before they reach any LLM.

## 4.1 Detected Secret Types

| Type | Pattern |
|------|---------|
| OpenSSH Private Keys | `-----BEGIN OPENSSH PRIVATE KEY-----` |
| PEM Private Keys | `-----BEGIN RSA PRIVATE KEY-----` |
| OpenAI API Keys | `sk-proj-...`, `sk-...` |
| AWS Access Keys | `AKIA...` (20 chars) |
| GitHub Tokens | `ghp_...`, `gho_...`, `ghu_...` |
| JWT Tokens | `eyJ...` (three base64 segments) |
| Bearer Tokens | `Bearer ...` (20+ char tokens) |
{.dense}

## 4.2 Secrets Configuration

```yaml
secrets_detection:
  enabled: true
  action: block  # block | redact | route_local
  entities:
    - OPENSSH_PRIVATE_KEY
    - PEM_PRIVATE_KEY
    - API_KEY_OPENAI
    - API_KEY_AWS
    - API_KEY_GITHUB
    - JWT_TOKEN
    - BEARER_TOKEN
  max_scan_chars: 200000
  log_detected_types: true
```

| Action | Behavior |
|--------|----------|
| **block** | Returns HTTP 400 error, request never reaches LLM |
| **redact** | Replaces secrets with placeholders, unredacts in response |
| **route_local** | Routes to local LLM when secrets detected (route mode only) |
{.dense}

> 
> The default action is `block`. Detected secrets are never logged in their original form.
{.is-danger}

# 5 · Dashboard & Authentication

PasteGuard includes a built-in monitoring dashboard accessible at `/dashboard`. To secure the dashboard, configure authentication in `config.yaml`:

```yaml
dashboard:
  auth:
    username: admin
    password: ${DASHBOARD_PASSWORD}
```

Set the `DASHBOARD_PASSWORD` environment variable or replace `${DASHBOARD_PASSWORD}` with your desired password.


# 6 · Language Support

PasteGuard supports 24 languages for PII detection: `ca`, `zh`, `hr`, `da`, `nl`, `en`, `fi`, `fr`, `de`, `el`, `it`, `ja`, `ko`, `lt`, `mk`, `nb`, `pl`, `pt`, `ro`, `ru`, `sl`, `es`, `sv`, `uk`.

Language is auto-detected per request. If a detected language isn't installed, PasteGuard falls back to English. The response header `X-PasteGuard-Language-Fallback: true` indicates when fallback is used.

> 
> Make sure the `languages` list in your `config.yaml` matches the `LANGUAGES` build argument used when building the Presidio container.
{.is-info}



# <img src="/youtube.png" class="tab-icon"> 7 · Video

*Video coming soon*