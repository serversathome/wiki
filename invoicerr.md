---
title: Invoicerr
description: A guide to deploying Invoicerr
published: true
date: 2025-12-11T17:14:47.625Z
tags: 
editor: markdown
dateCreated: 2025-12-11T16:48:55.711Z
---

# <img src="/invoicerr.png" class="tab-icon"> What is Invoicerr?
Invoicerr is a simple, open-source invoicing application designed to help freelancers manage their quotes and invoices efficiently. It provides a clean interface for creating, sending, and tracking quotes and invoices — so you get paid faster, with less hassle.

✨ Features
- Create and manage invoices
- Create and manage quotes (convertible to invoices)
- Manage clients and their contact details
- Track status of quotes and invoices (signed, paid, unread, etc.)
- Built-in quote signing system with secure tokens
- Generate and send quote/invoice emails directly from the app
- Generate clean PDF documents (quotes, invoices, receipts, and more)
- Custom brand identity: logo, company name, VAT, and more
- Authentication via JWT or OIDC (stored in cookies)
- International-friendly: Default English UI, customizable currencies
- SQLite database for quick local setup
- Docker & docker-compose ready for self-hosting
- Built with modern stack: React, NestJS, Prisma, SQLite/PostgreSQL
- REST API backend, ready for future integrations (mobile & desktop apps)
- Plugin system for community-made features



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Invoicerr
```yaml
services:
  invoicerr:
    image: ghcr.io/impre-visible/invoicerr:v1.4.0i
    ports:
      - 8001:80
    environment:
      - DATABASE_URL=postgresql://invoicerr:invoicerr@invoicerr_db:5432/invoicerr_db
      - APP_URL=http://10.99.0.242:8001
      - CORS_ORIGINS=http://10.99.0.242:8001,https://invoicerr.example.com
      # Required for email features
      - SMTP_HOST=smtp-relay.example.com
      - SMTP_USER="username@example.com"
      - SMTP_FROM="user-from@example.com" # Not required if SMTP_USER is the same as SMTP_FROM
      - SMTP_PASSWORD="your_smtp_password"
      - SMTP_PORT=587 # Change this to your SMTP port (default is 587 for TLS)
      - SMTP_SECURE=false # Set to true if your SMTP server requires a secure connection (default is false)
      # OIDC Configuration (example for Authentik)
      - OIDC_ISSUER="https://auth.chevrier.dev"
      - OIDC_NAME="Pocket ID"
      # Endpoints for OIDC
      - OIDC_AUTHORIZATION_ENDPOINT="https://auth.chevrier.dev/authorize"
      - OIDC_TOKEN_ENDPOINT="https://auth.chevrier.dev/token"
      - OIDC_USERINFO_ENDPOINT="https://auth.chevrier.dev/userinfo"
      - OIDC_TOKEN_REVOKE_ENDPOINT="https://auth.chevrier.dev/revoke"
      - OIDC_END_SESSION_ENDPOINT="https://auth.chevrier.dev/end_session"
      - OIDC_JWKS_URI="https://auth.chevrier.dev/.well-known/jwks.json"
      # Client ID and Secret for OIDC
      - OIDC_CLIENT_ID="your-client-id"
      - OIDC_CLIENT_SECRET="your-client-secret"
      # Optional, but recommended for docker deployments
      - JWT_SECRET="your_jwt_secret"
    depends_on:
      - invoicerr_db
  invoicerr_db:
    image: postgres:17
    environment:
      POSTGRES_USER: invoicerr
      POSTGRES_PASSWORD: invoicerr
      POSTGRES_DB: invoicerr_db
    volumes:
      - /mnt/tank/configs/invoicerr:/var/lib/postgresql/data
```