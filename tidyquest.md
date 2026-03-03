---
title: TidyQuest
description: A guide to deploying TidyQuest
published: true
date: 2026-03-03T18:07:39.432Z
tags: 
editor: markdown
dateCreated: 2026-03-03T18:07:39.432Z
---

# 🏠 What is TidyQuest?

**TidyQuest** is a self-hosted web application that gamifies household chores using RPG mechanics. Family members complete tasks to earn coins, build streaks, unlock achievements, and compete on a leaderboard — turning boring housework into an epic family adventure.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy TidyQuest

```yaml
services:
  tidyquest:
    image: mellowfox/tidyquest:latest
    container_name: tidyquest
    ports:
      - "3020:3000"
    environment:
      - NODE_ENV=production
      - JWT_SECRET=
    volumes:
      - /mnt/tank/configs/tidyquest/data:/app/data
    restart: unless-stopped
```

1. Generate a secure JWT secret before deploying:
   ```bash
   openssl rand -base64 32
   ```


# 2 · Configuration

## 2.1 User Roles

TidyQuest supports three user roles:

- **Admin** — Full access: manage users, create/edit/delete tasks and rooms, approve/reject reward requests, configure global settings, backup/restore data
- **Member** — Standard access: complete tasks, earn coins, redeem rewards, view leaderboard and achievements
- **Child** — Similar to member but reward requests require admin approval

## 2.2 Rooms & Tasks

Rooms organize your household tasks by location. TidyQuest ships with 8 default room types: Kitchen, Bedroom, Bathroom, Living Room, Office, Garage, Laundry, and Garden, along with 60+ predefined common tasks.

To create a room:
1. Go to the **Rooms** page
2. Click **Add Room** and select a room type
3. Choose from predefined tasks or create custom ones with frequencies from 1–365 days

Tasks have effort levels from 1–5, which determine coin rewards (5–25 coins per completion). When a task is completed, its health bar resets to 100% and begins decaying again based on its configured frequency.

## 2.3 Task Assignment Modes

When assigning tasks to specific users, admins can choose between three modes:

- **First** — The first person to complete the task earns all the coins
- **Shared** — Each assignee completes the task once and coins are split equally
- **Custom** — Define a custom coin percentage per assignee (e.g. 70% / 30%)

## 2.4 Rewards System

Admins create a reward catalog (TidyQuest includes 10 presets like movie night pick, extra game time, stay up late, etc.). Family members spend earned coins to redeem rewards. Admins can approve or reject requests — rejected requests refund the coins automatically.

## 2.5 Telegram Notifications (Optional)

1. Create a Telegram bot via [@BotFather](https://t.me/botfather)
2. Get your chat ID via [@userinfobot](https://t.me/userinfobot)
3. Configure in the app's **Settings** page (admin only)

Available notification types:
- **Daily Due Tasks** — Sent at a configured time (default 09:00)
- **Reward Requests** — Notifies admin when a member requests a reward
- **Achievements** — Celebrates unlocked achievements

## 2.6 Vacation Mode

Admins can enable vacation mode in **Settings** to pause all task health decay. This is useful during family trips so no one comes home to a dashboard full of red health bars.

## 2.7 Backup & Restore

**Via the UI:** Admin → Settings → Export Data → Download JSON. To restore: Admin → Settings → Import Data → Upload JSON.

