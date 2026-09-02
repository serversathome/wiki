---
title: NextDNS
description: A guide to configuring NextDNS
published: true
date: 2026-09-02T10:12:40.040Z
tags: 
editor: markdown
dateCreated: 2026-09-02T09:57:01.133Z
---

# <img src="/nextdns.png" class="tab-icon"> What is NextDNS?

**NextDNS** is a hosted DNS resolver with per-device profiles, category filtering, threat blocking and query logging. Think Pi-hole with a cloud control plane, plus the ability to apply a policy to a device that is not on your network.

That last part is the reason to run it alongside a local resolver rather than instead of one. A Pi-hole covers your LAN. A NextDNS profile installed on a phone covers that phone on cellular, at school, and on someone else's wifi.

Free tier is 300,000 queries per month with all features. Pro is $1.99/month or $19.90/year for unlimited queries.

> Want to say thanks? Sign up using [my link](https://nextdns.io/?from=s8235t86) and I get a small commission at no extra cost to you!
{.is-success}


> The free tier does not error when it hits the cap. It keeps resolving DNS and silently stops filtering and logging for the remainder of the month. A single household burns through 300k quickly, especially with the router pointed at it. Take the paid tier.
{.is-danger}

# 1 · Deploy NextDNS
# {.tabset}
## <img src="/apple-light.png" class="tab-icon"> iOS / iPadOS
**Setup** tab → **Setup Guide** → **iOS**, then scan the QR code with the phone's camera.
 
That single scan installs and configures the app together. It is the recommended path and it sidesteps the usual failure mode, where someone installs the app from the App Store, never enters a configuration ID, and ends up with encrypted DNS that filters nothing while looking exactly like it works.
 
> Scan the code from the profile you actually want applied. The Setup tab is profile-scoped, so a code generated while the Adults profile is selected will apply adult rules to the kid's phone. Check the profile dropdown in the top left before scanning.
{.is-warning}


## <img src="/android-robot.png" class="tab-icon"> Android

Two options:

1. **Private DNS** (Settings → Network → Private DNS) pointed at your DoT hostname from the Setup tab. No app, works on cellular, survives reboots.
2. The **NextDNS** app from Google Play, using your configuration ID.

Private DNS is the cleaner option where the device supports it.

## Router

**Setup** tab → **Setup Guide** → **Routers**, then pick your model.
 
Picking the right endpoint matters more than the router does. Use **DoH**, **DoT** or the **IPv6** addresses if your router supports any of them, because all three carry the config ID in the endpoint itself. Plain IPv4 does not, so it needs **Linked IP**, which ties your profile to your current public address.
 
> Linked IP breaks on a residential connection. A modem reboot or an ISP-side change gives you a new address, NextDNS stops recognising the queries, and filtering silently stops while the internet keeps working normally. Nobody notices for weeks.
{.is-danger}
 
If plain IPv4 is your only option, point the router's built-in **Dynamic DNS** at a free provider such as No-IP or DuckDNS, then enter that hostname under **Setup → Linked IP → Show advanced options**. NextDNS resolves it and keeps the link current. There is also a unique update URL there you can curl from a cron job on any always-on box.
 
Enable **per-device identification**, otherwise every client on the LAN collapses into one entry and the analytics stop being useful for spotting a device that should not be there.
 
> If an ISP gateway stays in the path, turn its wifi radio off. Two SSIDs in a house means one unfiltered SSID, and someone will find it. Then reboot the modem and re-check before you call the job done.
{.is-info}


## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  nextdns:
    image: ghcr.io/nextdns/nextdns:latest
    container_name: nextdns
    network_mode: host
    restart: unless-stopped
    command: run -config <your-config-id> -listen :53 -report-client-info
    volumes:
      - /mnt/tank/configs/nextdns:/etc/nextdns
```

Verify the current image tag and CLI flags against the upstream repo before deploying.

# 2 · Configuration

## 2.1 Subscribing

Billing lives on the **account**, not the profile. Every tab in the dashboard is profile-scoped, so the upgrade path is not where you will look for it.

1. Click your **email address** in the top right corner
2. Select **Account**
3. Click **Subscribe**

Card and PayPal both require a recurring payment method. Crypto is annual only.

## 2.2 Profiles

Create one profile per policy, not per device. The **Settings** tab has a `Duplicate` button, so build one profile properly and clone it as a template rather than reconfiguring from scratch.

## 2.3 Parental Control

| Setting | Value | Notes |
|---------|-------|-------|
| Categories | Porn, Gambling, Dating, Piracy | Piracy catches generic video hosts, so it is your most likely false-positive source |
| Websites, Apps & Games | Add by name | Blocks TikTok, Snapchat, Discord, Roblox individually |
| Enforce SafeSearch | On | Rewrites Google, Bing and DuckDuckGo into safe mode |
| Enforce YouTube Restricted Mode | On | Also hides comments |
| Block Bypass Methods | On | VPNs, proxies, Tor, third-party encrypted DNS |


**Block Bypass Methods** is the one that makes the rest hold. Without it, any VPN app walks around everything above.

**Recreation Time** allows a category or service during set hours rather than blocking it outright. The schedule is evaluated server side, so changing the clock on the client does nothing to it. No device-level time control can say the same.

## 2.4 Security

Most of this tab is on by default and should stay on. Two additions worth making on a restricted profile:

**Block Newly Registered Domains.** Domains under 30 days old are where phishing lives, and also where fast-rotating proxy and "unblocked" sites live. The most useful anti-bypass toggle on the page, and also the most likely to false-positive on a legitimately new site.

**Block Top-Level Domains.** No false-positive risk on the adult TLDs:

```
.xxx  .porn  .adult  .sex  .sexy  .webcam  .cam
.tk  .ml  .ga  .cf  .gq  .monster  .review
```

The second row is abuse-heavy free registrars rather than adult content.

> **Block Dynamic DNS Hostnames** is worth enabling but is your first suspect when something breaks, since plenty of consumer cameras and NVRs phone home over DDNS. It plus AI-Driven Threat Detection and NRDs are the three to toggle off first when diagnosing a complaint.
{.is-info}

## 2.5 Privacy

Start with the **NextDNS Ads & Trackers Blocklist** and stop there. Stacking five blocklists is how you end up debugging a broken checkout page on a Sunday.

**Block Disguised Third-Party Trackers** on. 

**Allow Affiliate & Tracking Links** off unless something legitimately breaks.

## 2.6 Settings

Three that people get wrong:

- **Enable Block Page** → **On**. With it off, a blocked query returns `0.0.0.0` and the client shows a generic connection failure. Nobody can distinguish a block from an outage, including you. Costs a little latency and an occasional HTTPS warning.
- **Bypass Age Verification** → **Off**. It does exactly what it says and defeats age gates on adult sites.
- **Enable Web3** → **Off**. NextDNS describes it as an unfiltered gateway to ENS, Handshake and IPFS. Those are not classic domains, so no category or blocklist applies. IPFS content is addressed by hash, so there is nothing to block even in principle.


# 3 · Verification

1. Load a domain in a blocked category. You should get the block page rather than a timeout.
2. Open the **Logs** tab and confirm queries are arriving from the expected client name.
3. On a phone, disconnect from wifi and repeat on cellular. If the queries stop appearing, the profile is not applying off-network and the deployment is wifi-only.

Expect thousands of queries per device per day. That is background app and telemetry traffic, not a problem.

# 4 · Limitations

- DNS filtering is domain level. It cannot see inside a TLS session, so it does not filter individual videos, read messages, or affect anything happening inside an allowed app.
- iCloud Private Relay routes Safari DNS to Apple and bypasses the resolver. Apple documents a supported network-side fix: return `NXDOMAIN` for `mask.icloud.com` and `mask-h2.icloud.com` rather than redirecting them.
- Since iOS 17, Safari Private Browsing sends DNS to Apple even with Private Relay off. Disabling Private Browsing via Screen Time closes that.
- On iOS the DNS setting can be switched back to Automatic in `Settings > General > VPN, DNS & Device Management` and no consumer restriction locks that screen. Supervising the device with Apple Configurator makes a profile non-removable, but supervision requires erasing and re-provisioning the phone.

Where you cannot get tamper-proof, aim for tamper-evident. With per-device identification enabled, a client that stops appearing in the logs entirely is unmistakable. A device generating three thousand queries a day and now generating zero has had its DNS changed.



# <img src="/youtube.png" class="tab-icon"> 5 · Video

