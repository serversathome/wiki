---
title: Financial Dashboard
description: A guide to deploy Financial Dashboard
published: true
date: 2026-08-05T20:38:18.977Z
tags: 
editor: markdown
dateCreated: 2026-08-05T20:38:18.977Z
---

# What is Firefly Dashboard?

**Firefly Dashboard** (`giorobert88/financial-dashboard`) is a mobile-first, responsive front end that plugs into an existing **Firefly III** instance and reframes your finances around a single question: how much can I actually spend right now? Rather than replacing Firefly III, it reads from it over the Core v1 API and layers on Safe-to-Spend pacing, a burn-rate chart against your previous cycle, and a fast queue for categorizing stray transactions.


**What it gives you:**

- **Safe-to-Spend** — takes your total asset balance and subtracts unpaid bills and credit card statements landing before your next payday
- **Burn comparison chart** — cumulative daily spend this cycle plotted against last cycle's pace
- **Uncategorized queue** — surfaces the last 7 days of uncategorized transactions at `/uncategorized`, with a badge count in the header; assigning a category writes straight back to Firefly III
- **Grouped ledger** — asset and liability accounts split into Current Accounts, Savings, and Credit Cards
- **Credit card rules** — per-card statement day, due day, and whether you pay the full statement balance or the minimum



# 1 · Prerequisites

You need a working **Firefly III** instance before any of this is useful. If you don't have one yet, Firefly III is available as a TrueNAS Community app (search "Firefly III" under **Apps**) or as a Docker container.

Once Firefly III is up, make sure of the following:

| Requirement | Where |
|-------------|-------|
| At least one active **asset account** | Accounts → Asset Accounts |
| A **Personal Access Token** | Options → Profile → OAuth → Personal Access Tokens |
| Credit cards have **Role** set to *Credit Card* | Edit the account in Firefly III |


> 
> An account is treated as a credit card if its role is set to **Credit Card**, or if its name contains the words "credit card". Setting the role properly is the cleaner option.
{.is-info}

Everything else — marking your primary current account, statement dates, payment preferences — gets configured in the dashboard UI itself. Those settings are stored as JSON inside the **notes field** of each Firefly III account, so they survive a container rebuild without needing a separate database.

# <img src="/docker.png" class="tab-icon"> 2 · Deploy Firefly Dashboard

## 2.1 Generate a session secret

Run this first and hold onto the output:

```bash
openssl rand -base64 32
```

## 2.2 Create the environment file

Create `/mnt/tank/configs/firefly-dashboard/.env.local`:

```bash
SESSION_SECRET="paste-the-openssl-output-here"

# Optional — both can be set in the UI instead
# FIREFLY_API_URL="http://192.168.1.10:8080"
# FIREFLY_PAT="your-firefly-personal-access-token"
```

## 2.3 Compose file

```yaml
services:
  firefly-dashboard:
    image: ghcr.io/giorobert88/financial-dashboard:latest
    container_name: firefly-dashboard
    restart: unless-stopped
    ports:
      - "3001:3000"
    env_file:
      - /mnt/tank/configs/firefly-dashboard/.env.local
    environment:
      - AUTH_FILE_PATH=/app/data/.dashboard_auth
    volumes:
      - /mnt/tank/configs/firefly-dashboard/data:/app/data
```


# 3 · Configuration

## 3.1 First launch

Browse to `http://your-server-ip:3001`. On the very first load you'll be prompted to create a **dashboard password**. This is separate from your Firefly III login and exists purely to gate access to the dashboard itself.



## 3.2 Connect to Firefly III

If you didn't pre-set the environment variables, go to **Settings → API Connection** and enter:

- Your Firefly III base URL (including port)
- Your Personal Access Token

Save, and the accounts ledger should populate immediately.

## 3.3 Payday and primary account

Under **Settings → Accounts**, mark which asset account is your primary current account. Safe-to-Spend calculates against your payday cycle, so this needs to be the account your salary actually lands in.

## 3.4 Credit card rules

For each credit card, set:

- **Statement day** — when the statement closes
- **Due day** — when payment is taken
- **Payment preference** — full statement balance or minimum payment

These drive what gets subtracted from Safe-to-Spend before your next payday. Setting them to "full statement balance" gives you the most conservative — and usually most honest — number.



## 3.5 Working the uncategorized queue

The **inbox icon** in the header shows a badge count of transactions from the last 7 days with no category. Tap through to `/uncategorized`, assign a category, and the row fades out as the update is written back to Firefly III. This is the main reason to keep the dashboard on your phone home screen — it turns categorization into a 30-second daily habit instead of a monthly slog.

