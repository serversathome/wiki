---
title: Scrutiny
description: A guide for deploying Scrutiny on TrueNAS and Docker
published: true
date: 2025-07-10T17:49:00.804Z
tags: 
editor: markdown
dateCreated: 2025-04-10T09:35:16.453Z
---

# ![](/scrutiny.png){class="tab-icon"}What is Scrutiny?
Scrutiny is a Hard Drive Health Dashboard & Monitoring solution, merging manufacturer provided S.M.A.R.T metrics with real-world failure rates.

# 1 · Deploy Scrutiny
# {.tabset}

## <img src="/truenas.png" class="tab-icon"> TrueNAS
Use all default values except for Storage Configuration, which needs **Host Path Configuration** for the `Config` folder at minimum.

> Permissions should be `Generic` for the `config` dataset since Scrutiny runs as root
{.is-info}

## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  scrutiny:
    container_name: scrutiny
    restart: unless-stopped
    image: ghcr.io/analogj/scrutiny:master-omnibus
    cap_add:
      - SYS_RAWIO
    ports:
      - 8083:8080 # webapp
      - 8086:8086 # influxDB admin
    volumes:
      - /run/udev:/run/udev:ro
      - /dev:/dev:ro
      - /mnt/tank/configs/scrutiny:/opt/scrutiny/config
      - /mnt/tank/configs/scrutiny/influxdb:/opt/scrutiny/influxdb
```

# 2 · Notifications
Scruitny can notify users in the event of value changes with a file added to the `config` folder. Add `scrutiny.yaml` to the `/config` directory with the following contents, uncommenting the notification type you would like:

```yaml
web:
  listen:
    port: 8080
    host: 0.0.0.0

    basepath: ''
  database:

    location: /opt/scrutiny/config/scrutiny.db
  src:
    frontend:
      path: /opt/scrutiny/web

  influxdb:
    host: 0.0.0.0
    port: 8086

    retention_policy: true


log:
  file: '' #absolute or relative paths allowed, eg. web.log
  level: INFO


# Notification "urls" look like the following. For more information about service specific configuration see
# Shoutrrr's documentation: https://containrrr.dev/shoutrrr/services/overview/
#
# note, usernames and passwords containing special characters will need to be urlencoded.
# if your username is: "myname@example.com" and your password is "124@34$1"
# your shoutrrr url will look like: "smtp://myname%40example%2Ecom:124%4034%241@ms.my.domain.com:587"

notify:
  urls:
#    - "discord://token@webhookid"
#    - "telegram://token@telegram?channels=channel-1[,channel-2,...]"
#    - "pushover://shoutrrr:apiToken@userKey/?priority=1&devices=device1[,device2, ...]"
#    - "slack://[botname@]token-a/token-b/token-c"
#    - "smtp://username:password@host:port/?fromAddress=fromAddress&toAddresses=recipient1[,recipient2,...]"
#    - "teams://token-a/token-b/token-c"
#    - "gotify://gotify-host/token"
#    - "pushbullet://api-token[/device/#channel/email]"
#    - "ifttt://key/?events=event1[,event2,...]&value1=value1&value2=value2&value3=value3"
#    - "mattermost://[username@]mattermost-host/token[/channel]"
#    - "ntfy://username:password@host:port/topic"
#    - "hangouts://chat.googleapis.com/v1/spaces/FOO/messages?key=bar&token=baz"
#    - "zulip://bot-mail:bot-key@zulip-domain/?stream=name-or-id&topic=name"
#    - "join://shoutrrr:api-key@join/?devices=device1[,device2, ...][&icon=icon][&title=title]"
#    - "script:///file/path/on/disk"
#    - "https://www.example.com/path"
```

## 2.1 Testing Notifications
1. Shell into the container
1. Execute `curl -X POST http://localhost:8080/api/health/notify`

# <img src="/youtube.png" class="tab-icon"> Video
[](https://youtu.be/5Pv2ip_v_2s)