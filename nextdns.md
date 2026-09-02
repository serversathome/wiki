---
title: NextDNS
description: A guide to configuring NextDNS
published: true
date: 2026-09-02T10:32:13.582Z
tags: 
editor: markdown
dateCreated: 2026-09-02T09:57:01.133Z
---

# <img src="/nextdns.png" class="tab-icon"> What is NextDNS?

**NextDNS** protects you from all kinds of security threats, blocks ads and trackers on websites and in apps and provides a safe and supervised Internet for kids — on all devices and on all networks.

Free tier is 300,000 queries per month with all features. Pro is $1.99/month or $19.90/year for unlimited queries.




> The free tier does not error when it hits the cap. It keeps resolving DNS and silently stops filtering and logging for the remainder of the month. A single household burns through 300k quickly, especially with the router pointed at it. Take the paid tier.
{.is-danger}

# 1 · Create your account

1. Go to [nextdns.io](https://nextdns.io/?from=s8235t86) and click **Try it now**

    > Want to say thanks? Sign up using [my link](https://nextdns.io/?from=s8235t86) and I get a small commission at no extra cost to you!
<!-- {blockquote:.is-success} -->

1. Sign up with an email address and password
1. You land on a dashboard with a configuration already created for you
1. Open the **Settings** tab and give it a name. `Kids` is a sensible first one

## 1.1 Subscribing

Billing lives on the **account**, not the profile. Every tab in the dashboard is profile-scoped, so the upgrade path is not where you will look for it.

1. Click your **email address** in the top right corner
2. Select **Account**
3. Click **Subscribe**

Card and PayPal both require a recurring payment method. Crypto is annual only.

> Subscribe before you deploy anything. A config that hits the free cap mid-month stops filtering with no error, so on a install for someone else you want the plan active before you walk out the door.
{.is-warning}

# 2 · Deploy NextDNS
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

Start by checking what the gateway actually is, because that decides everything else.
 
### If it is a stock ISP gateway (most homes)
 
Verizon, Xfinity, Cox and similar boxes do not run resolver software, will not accept a DoH URL, and often will not take custom IPv6 DNS even when the line carries IPv6. Some, notably Xfinity xFi, will not let you change DNS at all.
 
That leaves one workable option: **IPv4 with Linked IP**.
 
1. Open the gateway's admin page, usually `http://192.168.0.1` or `http://192.168.1.1`
2. Find the DNS settings and replace whatever is there with the two `45.90.x.x` servers from your **Linked IP** panel
3. Save, then click **Link IP** on the Setup tab
> Those IPv4 servers do not identify you. NextDNS matches queries to your profile by the public address they arrive from, so when your ISP hands out a new address after a modem reboot, filtering silently stops and the internet keeps working normally. Nobody notices for weeks.
{.is-danger}
 
**Do not leave it there.** Look for a **Dynamic DNS** section in the gateway. If it has one, point it at a free No-IP or DuckDNS hostname, then enter that hostname under **Setup → Linked IP → Show advanced options**. NextDNS resolves it and keeps the link current on its own.
 
If the gateway has no DDNS section, skip router setup entirely rather than leaving a link that will quietly die. Cover the phones and tablets individually instead. That is the coverage that matters anyway, it works on cellular, and it has none of this fragility. What you give up is the TV and the consoles, which is a small gap.


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

# 3 · Configuration

## 3.1 Profiles

Create one profile per policy, not per device. The **Settings** tab has a `Duplicate` button, so build one profile properly and clone it as a template rather than reconfiguring from scratch.

## 3.2 Parental Control

| Setting | Value | Notes |
|---------|-------|-------|
| Categories | Porn, Gambling, Dating, Piracy | Piracy catches generic video hosts, so it is your most likely false-positive source |
| Websites, Apps & Games | Add by name | Blocks TikTok, Snapchat, Discord, Roblox individually |
| Enforce SafeSearch | On | Rewrites Google, Bing and DuckDuckGo into safe mode |
| Enforce YouTube Restricted Mode | On | Also hides comments |
| Block Bypass Methods | On | VPNs, proxies, Tor, third-party encrypted DNS |


**Block Bypass Methods** is the one that makes the rest hold. Without it, any VPN app walks around everything above.

**Recreation Time** allows a category or service during set hours rather than blocking it outright. The schedule is evaluated server side, so changing the clock on the client does nothing to it. No device-level time control can say the same.

## 3.3 Security

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

## 3.4 Privacy

Start with the **NextDNS Ads & Trackers Blocklist** and stop there. Stacking five blocklists is how you end up debugging a broken checkout page on a Sunday.

**Block Disguised Third-Party Trackers** on. 

**Allow Affiliate & Tracking Links** off unless something legitimately breaks.

## 3.5 Settings

Three that people get wrong:

- **Enable Block Page** → **On**. With it off, a blocked query returns `0.0.0.0` and the client shows a generic connection failure. Nobody can distinguish a block from an outage, including you. Costs a little latency and an occasional HTTPS warning.
- **Bypass Age Verification** → **Off**. It does exactly what it says and defeats age gates on adult sites.
- **Enable Web3** → **Off**. NextDNS describes it as an unfiltered gateway to ENS, Handshake and IPFS. Those are not classic domains, so no category or blocklist applies. IPFS content is addressed by hash, so there is nothing to block even in principle.

## 3.6 Identify your devices

Turn this on or every client collapses into a single entry and the analytics stop being useful for the one thing they are best at, which is spotting a device that should not be there.

Instructions are at the bottom of the **Setup** tab. On DoT you prepend the name to the hostname (`Johns-iPhone-abc123.dns.nextdns.io`), on DoH you append it to the URL, and in the apps you enable **Send Device Name** in settings.


# 4 · Verification

1. Load a domain in a blocked category. You should get the block page rather than a timeout.
2. Open the **Logs** tab and confirm queries are arriving from the expected client name.
3. On a phone, disconnect from wifi and repeat on cellular. If the queries stop appearing, the profile is not applying off-network and the deployment is wifi-only.

Expect thousands of queries per device per day. That is background app and telemetry traffic, not a problem.

# 5 · Limitations

- DNS filtering is domain level. It cannot see inside a TLS session, so it does not filter individual videos, read messages, or affect anything happening inside an allowed app.
- iCloud Private Relay routes Safari DNS to Apple and bypasses the resolver. Apple documents a supported network-side fix: return `NXDOMAIN` for `mask.icloud.com` and `mask-h2.icloud.com` rather than redirecting them.
- Since iOS 17, Safari Private Browsing sends DNS to Apple even with Private Relay off. Disabling Private Browsing via Screen Time closes that.
- On iOS the DNS setting can be switched back to Automatic in `Settings > General > VPN, DNS & Device Management` and no consumer restriction locks that screen. Supervising the device with Apple Configurator makes a profile non-removable, but supervision requires erasing and re-provisioning the phone.

Where you cannot get tamper-proof, aim for tamper-evident. With per-device identification enabled, a client that stops appearing in the logs entirely is unmistakable. A device generating three thousand queries a day and now generating zero has had its DNS changed.



# <img src="/youtube.png" class="tab-icon"> 6 · Video