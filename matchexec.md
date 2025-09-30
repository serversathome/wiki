---
title: MatchExec
description: A guide to deploying MatchExec
published: true
date: 2025-09-30T14:07:12.942Z
tags: 
editor: markdown
dateCreated: 2025-09-30T14:07:12.942Z
---

# <img src="/matchexec.svg" class="tab-icon"> What is MatchExec?
MatchExec handles all aspects of managing video game matches. Do you want to host tournaments, organize game nights, or prove you're better than the other team? MatchExec is for you!

**Create a Match, and MatchExec handles the rest!**
✅ Full Discord Integration — Creates native Discord events, Rich detailed Embeds, Sign-up forms inside of Discord, announcements, reminders, and more
✅ Modern, Responsive Web UI — No matter the device or size, a beautiful, fast web interface awaits you
✅ No Timezone Issues — MatchExec shows you all times in your local timezone, and stores them in UTC. No need to worry about missing matches due to conversion errors
✅ Keep Score — Keep score of who wins each map and declare an overall winner
✅ Flexible — Support for different scoring types, custom modes, custom maps, whatever you want, it's playable
✅ Voice Announcers — 4 different personas to choose from: A evil queen, a British football announcer, a London radio DJ, and an American Wrestling Announcer

# <img src="/docker.png" class="tab-icon"> 1 · Deploy MatchExec
```yaml
services:
  matchexec:
    container_name: MatchExec
    restart: unless-stopped
    ports:
      - 3000:3000
    environment:
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/matchexec/data:/app/app_data
      - /mnt/tank/configs/matchexec/uploads:/app/public/uploads
    image: ghcr.io/slamanna212/matchexec:latest
```