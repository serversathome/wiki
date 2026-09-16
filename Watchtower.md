---
title: Watchtower
description: A guide on how to install Watchtower for container updates
published: true
date: 2026-09-16T10:43:12.370Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:01.108Z
---

# ![](/watchtower.png){class="tab-icon"} What is Watchtower?
Watchtower is a tool that automates the updating of Docker containers by pulling new images and restarting the containers with the same options used during deployment.

<div class="glance">
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Watchtower

```yaml
services:
  watchtower:
    image: nickfedor/watchtower
    container_name: watchtower
    environment:
      - TZ=America/New_York
      - WATCHTOWER_NOTIFICATIONS_HOSTNAME=
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_INCLUDE_STOPPED=true
      - WATCHTOWER_SCHEDULE=0 0 3 * * *
      # - WATCHTOWER_NOTIFICATION_URL=discord://token@webhookid
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

> I have added some custom environment variables to this compose file. For a full explanation of all possible variables, [see the docs](https://watchtower.nickfedor.com/v1.11.8/configuration/arguments/)
{.is-info}

> Watchtower does not update apps from the TrueNAS catalog. To skip those, add this line to the `environment` section with the container names:
> `- WATCHTOWER_DISABLE_CONTAINERS=`
{.is-warning}


# 2 · Run On-Command

If you ever need to update your apps outside of the specified schedule, use this command from the shell of the machine hosting watchtower:

```bash
docker exec -it watchtower /watchtower --run-once
```

# 3 · Notifications
 
Watchtower sends notifications through **Shoutrrr** URLs. Add a `WATCHTOWER_NOTIFICATION_URL` variable to the environment:
 
| Service | URL format |
|---------|------------|
| Discord | `discord://token@webhookid` |
| Gotify | `gotify://gotify.example.com/token` |
| ntfy | `ntfy://ntfy.sh/your-topic` |
| Email | `smtp://user:password@host:port/?from=you@example.com&to=you@example.com` |

 
```yaml
      - WATCHTOWER_NOTIFICATION_URL=discord://token@webhookid
```
 
# 4 · Useful Environment Variables
 
| Variable | What it does |
|----------|--------------|
| `WATCHTOWER_CLEANUP` | Removes the old image after updating so they do not pile up on disk |
| `WATCHTOWER_ROLLING_RESTART` | Restarts containers one at a time instead of all at once |
| `WATCHTOWER_INCLUDE_STOPPED` | Also checks stopped containers |
| `WATCHTOWER_REVIVE_STOPPED` | Starts stopped containers after updating them |
| `WATCHTOWER_RUN_ONCE` | Runs a single update check and then exits |
| `WATCHTOWER_DEBUG` | More detailed logging for troubleshooting |



