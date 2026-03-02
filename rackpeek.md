---
title: RackPeek
description: A guide to deploying RackPeek
published: true
date: 2026-03-02T18:36:35.268Z
tags: 
editor: markdown
dateCreated: 2026-03-02T18:36:35.268Z
---

# <img src="/rackpeek.png" class="tab-icon"> What is RackPeek?

**RackPeek** is a lightweight, opinionated CLI tool for documenting and managing home lab and small-scale IT infrastructure. It helps you track hardware, services, networks, and their relationships in a clear, scriptable, and reusable way — without enterprise bloat or proprietary lock-in. RackPeek stores all of its state in a simple YAML file, so your data is always portable and easy to inspect.

RackPeek supports managing servers, switches, routers, firewalls, access points, UPS units, desktops, laptops, and services — along with their components like CPUs, drives, GPUs, and NICs. It also includes a web UI for visualizing your infrastructure at a glance.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy RackPeek

```yaml
services:
  rackpeek:
    image: aptacode/rackpeek:latest
    container_name: rackpeek
    ports:
      - "8080:8080"
    volumes:
      - /mnt/tank/configs/rackpeek:/app/config
    restart: unless-stopped
```

# 2 · Configuration

## 2.1 Initial Setup

After deploying RackPeek, you can begin documenting your infrastructure immediately through the web UI or the CLI tool (`rpk`). There is no database to configure — everything is stored in the `config.yaml` file mapped to your config volume.

## 2.2 CLI Usage

RackPeek uses the `rpk` command with a tree-style syntax. Here are some common operations:

**View a summary of all resources:**
```bash
rpk summary
```

**Add a server:**
```bash
rpk servers add
```

**Describe a specific server:**
```bash
rpk servers describe <server-name>
```

**View the dependency tree of a server:**
```bash
rpk servers tree <server-name>
```

**Manage hardware components:**
```bash
rpk servers cpu add <server-name>
rpk servers drive add <server-name>
rpk servers gpu add <server-name>
rpk servers nic add <server-name>
```

## 2.3 Resource Types

RackPeek can track the following infrastructure types:

| Resource | Command | Description |
|----------|---------|-------------|
| Servers | `rpk servers` | Rack/tower servers with CPU, drive, GPU, and NIC components |
| Switches | `rpk switches` | Network switches with port management |
| Routers | `rpk routers` | Network routers with port management |
| Firewalls | `rpk firewalls` | Firewall appliances with port management |
| Access Points | `rpk accesspoints` | Wireless access points |
| UPS | `rpk ups` | Uninterruptible power supplies |
| Desktops | `rpk desktops` | Desktop computers with full hardware tracking |
| Laptops | `rpk laptops` | Laptops with full hardware tracking |
| Services | `rpk services` | Software services and their configurations |
| Systems | `rpk systems` | Logical systems and dependency tracking |
{.dense}

## 2.4 Web UI

RackPeek includes a built-in web interface that provides a visual overview of your entire infrastructure. Access it at the port you configured (default `8080`). The web UI allows you to browse resources, view relationships, and get a quick summary of your lab without touching the CLI.

> 
> The web UI is read/write — you can manage your infrastructure from either the CLI or the browser.
{.is-success}

# <img src="/youtube.png" class="tab-icon"> 3 · Video

Coming soon!