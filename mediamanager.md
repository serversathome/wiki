---
title: Media Manager
description: A guide to deploying Media Manager via docker
published: true
date: 2025-08-04T11:59:15.707Z
tags: 
editor: markdown
dateCreated: 2025-08-04T11:59:15.707Z
---

# ![](/mediamanager.png){class="tab-icon"} What is Media Manager?
MediaManager is modern software to manage your TV and movie library. It is designed to be a replacement for Sonarr, Radarr, Overseer, and Jellyseer. It supports TVDB and TMDB for metadata, supports OIDC and OAuth 2.0 for authentication and supports Prowlarr and Jackett. 

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Media Manager
```yaml
services:
  mediamanager:
    image: ghcr.io/maxdorninger/mediamanager/mediamanager:latest
    container_name: mediamanager
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      - CONFIG_DIR=/app/config
    volumes:
      - /mnt/tank/media:/data/
      - /mnt/tank/configs/mediamanager:/app/config/
      - /mnt/tank/configs/mediamanager/images/:/data/images/
      
  db:
    image: postgres:latest
    restart: unless-stopped
    container_name: mediamanager-db
    volumes:
      - /mnt/tank/configs/mediamanager/postgres:/var/lib/postgresql/data
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}" ]
      interval: 10s
      timeout: 5s
      retries: 5
    environment:
      POSTGRES_USER: MediaManager
      POSTGRES_DB: MediaManager
      POSTGRES_PASSWORD: MediaManager

```

Once this is created you need to paste the toml file below to `/mnt/tank/configs/mediamanager` using something like `nano` by entering this command in the TrueNAS shell:
```bash
nano /mnt/tank/configs/mediamanager/config.toml
```

<details>
<summary><strong>Show me the code!</strong> (click to expand)</summary>
  
```toml
# MediaManager Example Configuration File
# This file contains all available configuration options for MediaManager
# Documentation: https://maxdorninger.github.io/MediaManager/introduction.html
#
# This is an example configuration file that gets copied to your config folder
# on first boot. You should modify the values below to match your setup.

[misc]
# it's very likely that you need to change this for MediaManager to work
frontend_url = "http://localhost:8000/web/" # note the trailing slash
cors_urls = ["http://localhost:8000"] # note the lack of a trailing slash

image_directory = "/data/images"
tv_directory = "/data/tv"
movie_directory = "/data/movies"
torrent_directory = "/data/downloads" # this is where MediaManager will search for the downloaded torrents and usenet files

# you probaly don't need to change this
development = false

# Custom Media Libraries
# These paths should match your volume mounts in docker-compose.yaml
# Example: if you mount "./movies:/media/movies" then use path = "/media/movies/subdirectory"
[[misc.tv_libraries]]
name = "Live Action"
path = "/data/tv/"  # Change this to match your actual TV shows location

[[misc.movie_libraries]]
name = "Documentary"
path = "/data/movies/"  # Change this to match your actual movies location

[database]
host = "db"
port = 5432
user = "MediaManager"
password = "MediaManager"
dbname = "MediaManager"

[auth]
email_password_resets = false # if true, you also need to set up SMTP (notifications.smtp_config)

token_secret = "bc597497f012ccb883ef93aead6881fe2d988965c4cfff25832486e3a3c73b82" # generate a random string with "openssl rand -hex 32", e.g. here https://www.cryptool.org/en/cto/openssl/
session_lifetime = 86400  # this is how long you will be logged in after loggin in, in seconds

# Admin users: Users who register with these email addresses will automatically become administrators
# If no users exist in the database, a default admin user will be created with the first email in this list
admin_emails = ["admin@example.com", "admin2@example.com"]

# OpenID Connect settings
[auth.openid_connect]
enabled = false
client_id = ""
client_secret = ""
configuration_endpoint = "https://openid.example.com/.well-known/openid-configuration"
name = "OpenID"

[notifications]
# SMTP settings for email notifications and email password resets
[notifications.smtp_config]
smtp_host = "smtp.example.com"
smtp_port = 587
smtp_user = "admin"
smtp_password = "admin"
from_email = "mediamanager@example.com"
use_tls = true

# Email notification settings
[notifications.email_notifications]
enabled = false
emails = ["admin@example.com", "admin2@example.com"]  # List of email addresses to send notifications to

# Gotify notification settings
[notifications.gotify]
enabled = false
api_key = ""
url = "https://gotify.example.com"

# Ntfy notification settings
[notifications.ntfy]
enabled = false
url = "https://ntfy.sh/your-topic"

# Pushover notification settings
[notifications.pushover]
enabled = false
api_key = ""
user = ""

[torrents]
# qBittorrent settings
[torrents.qbittorrent]
enabled = false
host = "http://localhost"
port = 8080
username = "admin"
password = "admin"

# Transmission settings
[torrents.transmission]
enabled = false
username = "admin"
password = "admin"
https_enabled = true
host = "localhost"
port = 9091
path = "/transmission/rpc" # RPC request path target, usually "/transmission/rpc"

# SABnzbd settings
[torrents.sabnzbd]
enabled = false
host = "http://localhost"
port = 8080
api_key = ""
base_path = "/api"

[indexers]
# Prowlarr settings
[indexers.prowlarr]
enabled = false
url = "http://localhost:9696"
api_key = ""

# Jackett settings
[indexers.jackett]
enabled = false
url = "http://localhost:9117"
api_key = ""
indexers = ["1337x", "torrentleech"]  # List of indexer names to use

# Title-based scoring rules
[[indexers.title_scoring_rules]]
name = "prefer_h265"
keywords = ["h265", "hevc", "x265", "h.265", "x.265"]
score_modifier = 100
negate = false

[[indexers.title_scoring_rules]]
name = "avoid_cam"
keywords = ["cam", "ts"]
score_modifier = -10000
negate = false

# Indexer flag-based scoring rules
[[indexers.indexer_flag_scoring_rules]]
name = "reject_non_freeleech"
flags = ["freeleech", "freeleech75"]
score_modifier = -10000
negate = true

[[indexers.indexer_flag_scoring_rules]]
name = "reject_nuked"
flags = ["nuked"]
score_modifier = -10000
negate = false

# Scoring rulesets
[[indexers.scoring_rule_sets]]
name = "default"
libraries = ["ALL_TV", "ALL_MOVIES"]
rule_names = ["prefer_h265", "avoid_cam", "reject_nuked"]

[[indexers.scoring_rule_sets]]
name = "strict_quality"
libraries = ["ALL_MOVIES"]
rule_names = ["prefer_h265", "avoid_cam", "reject_non_freeleech"]

# its very unlikely that you need to change this
[metadata]
[metadata.tmdb]
tmdb_relay_url = "https://metadata-relay.maxid.me/tmdb"

[metadata.tvdb]
tvdb_relay_url = "https://metadata-relay.maxid.me/tvdb"
```
  </details>