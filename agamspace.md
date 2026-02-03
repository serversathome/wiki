---
title: Agam Space
description: A guide to deploying Agam Space
published: true
date: 2026-02-03T19:12:40.839Z
tags: 
editor: markdown
dateCreated: 2026-02-03T14:03:10.655Z
---

# <img src="/agam-space.png" class="tab-icon"> What is Agam Space?

**Agam Space** is a self-hosted, zero-knowledge, end-to-end encrypted file storage platform. All files and metadata are encrypted in your browser before upload using XChaCha20-Poly1305 encryption. Your master password never leaves your device, and the server stores only encrypted blobs — even the server admin cannot access your files.

Think of it as a self-hosted alternative to Mega or Proton Drive, where privacy is guaranteed by cryptography rather than trust.

> 
> Agam Space is in **early beta** and not ready for production use. Bugs and data loss are possible. Do not use as your only backup.
{.is-warning}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Agam Space

```yaml
services:
  agam-space:
    image: agamspace/agam-space:latest
    container_name: agam-space
    environment:
      - DATABASE_HOST=agam-db
      - DATABASE_PORT=5432
      - DATABASE_USERNAME=agam
      - DATABASE_PASSWORD=changeme
      - DATABASE_NAME=agam
      - HTTP_PORT=3331
      - ALLOW_NEW_SIGNUP=true
    restart: unless-stopped
    ports:
      - "3331:3331"
    volumes:
      - /mnt/tank/configs/agamspace/data:/data
    depends_on:
      - agam-db

  agam-db:
    image: postgres:15-alpine
    container_name: agam-db
    environment:
      - POSTGRES_USER=agam
      - POSTGRES_PASSWORD=changeme
      - POSTGRES_DB=agam
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/agamspace/postgres:/var/lib/postgresql/data
```

1. Change the `POSTGRES_PASSWORD` to something secure
2. Update the `DATABASE_URL` password to match
> 
> For production deployments, place Agam Space behind a reverse proxy with HTTPS. The encryption happens client-side, but you still want secure transport.
{.is-info}

# 2 · Configuration

## 2.1 Initial Setup

On first access, you'll create an **Owner** account. This is the admin account that can manage users and quotas.

Next, you'll set up your **Master Password**. This password:
- Derives your Cryptographic Master Key (CMK) using Argon2id
- Never leaves your device
- Cannot be recovered if lost — there is no password reset

> 
> **Critical:** If you lose your master password, your data is unrecoverable. The server cannot decrypt your files.
{.is-danger}

## 2.2 Biometric Unlock

After initial setup, you can enable **WebAuthn biometric unlock** on trusted devices:
- Touch ID (Mac)
- Face ID (iOS)
- Windows Hello
- Hardware security keys

This stores an encrypted copy of your CMK on the device, allowing quick unlock without typing your master password each time.

## 2.3 User Management

As the Owner, you can:
- Create additional users (Admin or regular roles)
- Set per-user storage quotas
- Manage user accounts

## 2.4 SSO Integration

Agam Space supports SSO via OpenID Connect with:
- Authelia
- Authentik
- Keycloak
- PocketID

Refer to the [official documentation](https://docs.agamspace.app/configuration/) for SSO configuration details.

| Environment Variable | Description |
|---------------------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
{.dense}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

*No video yet — check back soon!*