---
title: Invoicerr
description: A guide to deploying Invoicerr via docker
published: true
date: 2025-08-04T11:29:50.007Z
tags: 
editor: markdown
dateCreated: 2025-08-04T11:29:50.007Z
---

# ![](/invoicerr.png){class="tab-icon"} What is Invoicerr?
Invoicerr is a simple, open-source invoicing application designed to help freelancers manage their quotes and invoices efficiently. It provides a clean interface for creating, sending, and tracking quotes and invoices — so you get paid faster, with less hassle.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Invoicerr
```yaml
services:
  invoicerr:
    image: ghcr.io/impre-visible/invoicerr:latest
    container_name: invoicerr
    ports:
      - "8001:80"
    environment:
      - DATABASE_URL=postgresql://invoicerr:invoicerr@invoicerr_db:5432/invoicerr_db
      - APP_URL=https://invoicerr.example.com # Required for email templates, as it redirects to the app
      - CORS_ORIGINS=http://localhost:5173,https://invoicerr.example.com # Comma-separated list of allowed origins for CORS

      # Required for email features
      - SMTP_HOST=smtp-relay.example.com
      - SMTP_USER="username@example.com"
      - SMTP_FROM="user-from@example.com" # Not required if SMTP_USER is the same as SMTP_FROM
      - SMTP_PASSWORD="your_smtp_password"
      - SMTP_PORT=587 # Change this to your SMTP port (default is 587 for TLS)
      - SMTP_SECURE=false # Set to true if your SMTP server requires a secure connection (default is false)

      # OIDC Configuration (example for Authentik)
      - OIDC_ISSUER="https://auth.chevrier.dev"
      - # Endpoints for OIDC
      - OIDC_AUTHORIZATION_ENDPOINT="https://auth.chevrier.dev/authorize"
      - OIDC_TOKEN_ENDPOINT="https://auth.chevrier.dev/token"
      - OIDC_USERINFO_ENDPOINT="https://auth.chevrier.dev/userinfo"
      - OIDC_TOKEN_REVOKE_ENDPOINT="https://auth.chevrier.dev/revoke"
      - OIDC_END_SESSION_ENDPOINT="https://auth.chevrier.dev/end_session"
      - OIDC_JWKS_URI="https://auth.chevrier.dev/.well-known/jwks.json"
      - # Callback URL for OIDC
      - OIDC_CALLBACK_URL="https://invoicerr.chevrier.dev/api/auth/callback"
      - # Client ID and Secret for OIDC
      - OIDC_CLIENT_ID="your-client-id"
      - OIDC_CLIENT_SECRET="your-client-secret"

      - VITE_OIDC_ENDPOINT=https://invoicerr.chevrier.dev/api/auth/oidc/login # If present, display OIDC login option in the frontend

      # Optional, but recommended for docker deployments
      - JWT_SECRET="your_jwt_secret" # Used for JWT authentication, can be any random string
    depends_on:
      - invoicerr_db

  invoicerr_db:
    image: postgres:latest
    container_name: invoicerr_db
    environment:
      POSTGRES_USER: invoicerr
      POSTGRES_PASSWORD: invoicerr
      POSTGRES_DB: invoicerr_db
    volumes:
      - /mnt/tank/configs/invoicerr:/var/lib/postgresql/data
```