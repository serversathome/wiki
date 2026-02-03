---
title: Nginx UI
description: A guide to deplpying Nginx UI
published: true
date: 2026-02-03T13:56:54.129Z
tags: 
editor: markdown
dateCreated: 2026-02-03T13:56:54.129Z
---

# <img src="/nginx-ui.png" class="tab-icon"> What is Nginx UI?

**Nginx UI** is a comprehensive web-based interface for managing Nginx servers. It provides a modern dashboard for editing configurations, managing SSL certificates, viewing logs, and monitoring server performance — all without touching the command line.

**Key Features:**
- Online statistics for CPU, memory, load average, and disk usage
- Automatic configuration backup with version comparison and restore
- One-click Let's Encrypt certificate deployment with auto-renewal
- NgxConfigEditor block editor or Ace Code Editor with syntax highlighting
- Built-in ChatGPT assistant for configuration help (supports multiple models including Deepseek-R1)
- LLM-powered code completion in the editor
- Online Nginx log viewer
- Web terminal for server access
- Cluster management for multi-server environments
- 2FA authentication support
- Dark mode and responsive design

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Nginx UI

```yaml
services:
  nginx-ui:
    image: uozi/nginx-ui:latest
    container_name: nginx-ui
    environment:
      - TZ=America/New_York
    restart: unless-stopped
    ports:
      - "8080:80"
      - "8443:443"
    volumes:
      - /mnt/tank/configs/nginx:/etc/nginx
      - /mnt/tank/configs/nginx-ui:/etc/nginx-ui
      - /var/run/docker.sock:/var/run/docker.sock
```

1. Ensure the `/mnt/tank/configs/nginx` directory is empty on first run
2. Deploy the stack and navigate to `http://your-server-ip:8080/install`
3. Complete the initial setup wizard to create your admin account

> 
> The Nginx UI image includes Nginx itself. Ports 8080 and 8443 map to the container's port 80 and 443, allowing you to use Nginx UI as your primary reverse proxy.
{.is-info}

# 2 · Configuration

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `TZ` | Timezone for the container | UTC |
{.dense}

## Volume Mounts

| Path | Description |
|------|-------------|
| `/etc/nginx` | Nginx configuration directory (must be empty on first run) |
| `/etc/nginx-ui` | Nginx UI data and settings |
| `/var/run/docker.sock` | Optional: enables Docker container management |
{.dense}

## Nginx Configuration Structure

Nginx UI follows the Debian-style configuration layout. Your `nginx.conf` should include:

```nginx
http {
    # ...
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

Site configurations are stored in `sites-available` and symlinked to `sites-enabled` when activated.

## Default Ports

| Port | Description |
|------|-------------|
| 80 | HTTP (Nginx) |
| 443 | HTTPS (Nginx) |
| 9000 | Nginx UI panel (when running standalone) |
{.dense}


# 3 · Resources

- **GitHub:** https://github.com/0xJacky/nginx-ui
- **Documentation:** https://nginxui.com/
- **Demo:** https://demo.nginxui.com (admin/admin)

# <img src="/youtube.png" class="tab-icon"> 4 · Video

*No video yet — check back soon!*