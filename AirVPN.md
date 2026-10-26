---
title: AirVPN
description: Instructions on how to use AirVPN to create a wireguard config for torrenting
published: true
date: 2026-10-26T10:35:38.236Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:16.582Z
---

![](/sky_1010x227.png)
# What is AirVPN?
An OpenVPN and WireGuard based VPN operated by activists in defense of net neutrality, privacy and against censorship.

This impenetrable tunnel prevents criminal organizations, your ISP or even your government from spying on your communications.


> Want to say thanks? Sign up using [my link](https://airvpn.org/?referred_by=669318) and I get free VPN time!
{.is-success}

# 1 · Creating a .conf File

To use AirVPN, sign up and generate a Wireguard Configuration from the **Client Area** tab, then the **Config Generator** box (_not_ the Eddie client).

![](https://wiki.hydrology.cc/airvpnconf.png)

1. Click the **Advanced** slider
1. Change the **IP Exit Layer** to **IPv4 Only**
1. Switch the **Wireguard** slide
1. Scroll down the the single server section and select one which is fastest (usually the ones which are 20000 MB/s)
1. Scroll all the way to the bottom and click **Generate**
1. Once it saves the file, rename it **wg0.conf.**

> You will need to copy these contents to the folder housing your qBittorrent config
{.is-warning}


# 2 · Generating a .conf File with a Script

Instead of using the Config Generator by hand, you can let my `airvpn-wg.sh` script build the `wg0.conf` for you straight from the AirVPN API. It picks the best server for you, and it can also **send you alerts** and **automatically switch servers** when the one you're on goes down. My [\*arr stack script](https://wiki.serversatho.me/Folder-Structure#h-21-running-the-script) offers it when it asks for qBittorrent's WireGuard config (option 2, **Auto-generate**), so if you use that script you don't need to run this one separately.

> You need an AirVPN **API key**, which requires a premium account. Create one at [airvpn.org/apisettings](https://airvpn.org/apisettings/).
{.is-info}

## 2.1 Running the Script
To view the script, [look here](https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/airvpn-wg.sh). To run it, execute this one-liner in the TrueNAS shell:

```bash
sudo su -c "wget -O airvpn-wg.sh https://raw.githubusercontent.com/serversathome/ServersatHome/refs/heads/main/airvpn-wg.sh && chmod +x airvpn-wg.sh && bash airvpn-wg.sh"
```

The script walks you through these steps:

1. **API key**: paste your AirVPN API key. It is only saved when the monitor needs it later (auto-switch, or device-offline / expiry alerts); otherwise you are asked again next time.
1. **Device**: pick one of the devices (keys) on your AirVPN account, or create a new one.
1. **Server**: choose your preferred countries and a minimum server bandwidth (by default the largest size AirVPN offers, currently 20000 Mbit/s). The script lists only healthy servers and ranks them by load, reliability (outages over the last 90 days for the top 5, 10 days for the rest) and measured ping. Every figure is shown, so you can still pick any server you like.
1. **Generate**: it writes the `wg0.conf` (WireGuard, Linux, IPv4 only), saves the previous one as `wg0.conf.bak`, and checks that the config really belongs to the device you picked. Any running container that uses this `wg0.conf` (like qBittorrent) is restarted so it switches over.
1. **Alerts** (optional): pick a channel (**ntfy**, **Discord**, **Gotify** or **Telegram**) and which events you want to hear about.
1. **Auto-switch** (optional): when your server goes down, the script rewrites `wg0.conf` for the best healthy server that matches the same countries and bandwidth, then restarts qBittorrent.

## 2.2 Alerts and Auto-Switch

You can turn on any of these alerts:

| Alert | Default |
| --- | --- |
| Server down (health error, or removed by AirVPN) | On |
| Server warning (e.g. packet loss, maintenance announced) | On |
| Server load at or above a threshold (90% by default) | Off |
| Your device is not connected to AirVPN | Off |
| Subscription expires within a number of days (7 by default) | On |
| All clear when a problem goes away | On |

Each problem is alerted once when it starts, and you get an "all clear" when it goes away.

When alerts or auto-switch are on, the script adds a cron job that runs `airvpn-wg.sh monitor` every **5 minutes**. On TrueNAS it shows up under **System > Advanced Settings > Cron Jobs** as **AirVPN monitor**. If an auto-switch fails (for example, no server matches your filters), it is retried every 30 minutes.

> The script keeps a copy of itself, plus its settings (`airvpn-wg.env`, readable by root only), next to your `wg0.conf`. The cron job runs that copy, so the one you downloaded is removed once it has been moved there.
{.is-info}

## 2.3 Changing Settings Later

Run the copy next to your `wg0.conf` with one of these commands:

| Command | What it does |
| --- | --- |
| `bash airvpn-wg.sh --output <path>/wg0.conf` | Pick a different device or server and generate a new config |
| `bash airvpn-wg.sh alerts --output <path>/wg0.conf` | Change only the alert channel, events and auto-switch |
| `bash airvpn-wg.sh test-alert --output <path>/wg0.conf` | Send a test message to your alert channel |
| `bash airvpn-wg.sh monitor --output <path>/wg0.conf` | Run one check by hand (what the cron job does) |
| `bash airvpn-wg.sh --help` | Show every option |

You can also skip the menus with `--device <name>` and `--server <name>` (for example `--server Menkent`), and choose the WireGuard port with `--port` (`1637` by default, or `47107` / `51820`).

> AirVPN bans IPs that send more than 600 API requests in 10 minutes, so the script limits itself to 300 per run. Building the server list usually takes 40 to 120 requests, and a monitor check takes 1 to 3.
{.is-warning}

# 3 · Port Forwarding

The point of using AirVPN is the port forward for qBittorrent. Go the **Client Area** tab, then the **Ports** box, and click **Manage**.

![](/screenshot_from_2024-02-23_09-42-58.png)

1. At the bottom, click **Request a new port**
1. Change the **Device** name to the device which you just created the wireguard config file for
1. Change the **IP Layer** to **IPv4 Only**
1. After you add this to your TrueNAS and qBit configuration, come back and click **Test open**. It should look like this if its working:

![](https://wiki.hydrology.cc/portforward.png)

# 4 · Script Video

