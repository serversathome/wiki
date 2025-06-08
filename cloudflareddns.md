---
title: Cloudflare DDNS
description: A guide to installing Cloudflare DDNS Updater in Docker
published: true
date: 2025-06-08T18:39:35.705Z
tags: 
editor: markdown
dateCreated: 2024-06-14T20:14:22.604Z
---

![](/cloudflare.png)

# What is Cloudflared DDNS?
A feature-rich and robust Cloudflare DDNS updater with a small footprint. The program will detect your machine's public IP addresses and update DNS records using the Cloudflare API.

# Docker Compose

```yaml
services:
  cloudflare-ddns:
    image: favonia/cloudflare-ddns:latest
    network_mode: host
    # This bypasses network isolation and makes IPv6 easier (optional; see below)
    restart: unless-stopped
    user: "1000:1000"
    read_only: true
    # Make the container filesystem read-only (optional but recommended)
    cap_drop: [all]
    # Drop all Linux capabilities (optional but recommended)
    security_opt: [no-new-privileges:true]
    # Another protection to restrict superuser privileges (optional but recommended)
    environment:
      - CLOUDFLARE_API_TOKEN=
      - DOMAINS=
      - PROXIED=true
      - IP6_PROVIDER=none
```


| Field | Value |
| --- | --- |
| Cloudflare API Token | The value of `CLOUDFLARE_API_TOKEN` should be an API token (not an API key), which can be obtained from the [API Tokens page](https://dash.cloudflare.com/profile/api-tokens). Use the **Edit zone DNS** template to create a token. The less secure API key authentication is deliberately not supported. There is an optional feature (available since version 1.14.0) that lets you maintain a [WAF list](https://developers.cloudflare.com/waf/tools/lists/custom-lists/) of detected IP addresses. To use this feature, edit the token and grant it the **Account - Account Filter Lists - Edit** permission. If you only need to update WAF lists, not DNS records, you can remove the **Zone - DNS - Edit** permission. Refer to the detailed documentation below for information on updating WAF lists. | 
| Domains | The value of DOMAINS should be a list of fully qualified domain names (FQDNs) separated by commas. For example, `DOMAINS=example.org,www.example.org,example.io` instructs the updater to manage the domains `example.org`, `www.example.org`, and `example.io`. These domains do not have to share the same DNS zone---the updater will take care of the DNS zones behind the scene. |
| Proxied=true |  The setting `PROXIED=true` instructs Cloudflare to cache webpages and hide your IP addresses. If you wish to bypass that and expose your actual IP addresses, remove `PROXIED=true`. If your traffic is not HTTP(S), then Cloudflare cannot proxy it and you should probably turn off the proxying by removing `PROXIED=true`. The default value of `PROXIED` is false. |
| IP6_PROVIDER=none | The updater, by default, will attempt to update DNS records for both IPv4 and IPv6, and there is no harm in leaving the automatic detection on even if your network does not work for one of them. However, if you want to disable IPv6 entirely (perhaps to avoid seeing the detection errors), add `IP6_PROVIDER=none`.  |



> You need to add you Cloudflare token to the line `CF_API_TOKEN=` which can be found by going to your Cloudflare dashboard, clicking on **Websites** > go to the domain you purchased > API (in the bottom right)
{.is-warning}


![](/screenshot_from_2024-06-14_16-04-24.png)

You want to create a new API Token from template to **Edit Zone DNS**:

![](/screenshot_from_2024-06-14_16-07-41.png)

![](/screenshot_from_2024-06-14_16-09-46.png)

Then change the **Zone Resources** section to include **All Zones**, then hit the blue button at the bottom to **Continue to Summary**, then **Create Token**. 
> **Save that token** because you won't be shown it again!
{.is-danger}

# ❓ Frequently Asked Questions
## I simulated an IP address change by editing the DNS records, but the updater never picked it up!

Please rest assured that the updater is working as expected. It will update the DNS records immediately for a real IP change. Here is a detailed explanation. There are two causes of an IP mismatch:

- A change of your actual IP address (a real change), or
- A change of the IP address in the DNS records (a simulated change).

The updater assumes no one will actively change the DNS records. In other words, it assumes simulated changes will not happen. It thus caches the DNS records and cannot detect your simulated changes. However, when your actual IP address changes, the updater will immediately update the DNS records. Also, the updater will eventually check the DNS records and detect simulated changes after `CACHE_EXPIRATION` (six hours by default) has passed.

If you really wish to test the updater with simulated IP changes in the DNS records, you can set `CACHE_EXPIRATION=1ns` (all cache expiring in one nanosecond), effectively disabling the caching. However, it is recommended to keep the default value (six hours) to reduce your network traffic.

## How can I see the timestamps of the IP checks and/or updates?

The updater does not itself add timestamps because all major systems already timestamp everything:

- If you are using Docker Compose, Kubernetes, or Docker directly, add the option --timestamps when viewing the logs.
- If you are using Portainer, [enable “Show timestamp” when viewing the logs](https://docs.portainer.io/user/docker/containers/logs).

## Why did the updater detect a public IP address different from the WAN IP address on my router?

Is your “public” IP address on your router between 100.64.0.0 and 100.127.255.255? If so, you are within your ISP’s [CGNAT (Carrier-grade NAT)](https://en.wikipedia.org/wiki/Carrier-grade_NAT). In practice, there is no way for DDNS to work with CGNAT, because your ISP does not give you a real public IP address, nor does it allow you to forward IP packages to your router using cool protocols such as [Port Control Protocol](https://en.wikipedia.org/wiki/Port_Control_Protocol). You have to give up DDNS or switch to another ISP. You may consider other services such as [Cloudflare Tunnel](/CloudflareTunnels)https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) that can work around CGNAT.


## Help! I got exec /bin/ddns: operation not permitted

Certain Docker installations may have issues with the no-new-privileges security option. If you cannot run Docker images with this option (including this updater), removing it might be necessary. This will slightly compromise security, but it’s better than not running the updater at all. If only this updater is affected, please [report this issue on GitHub](https://github.com/favonia/cloudflare-ddns/issues/new).

## I am getting error code: 1034

We have received reports of recent issues with the default IP provider, `cloudflare.trace`. Some users are encountering an "error code: 1034," likely due to internal problems with Cloudflare's servers. To work around this, please upgrade the updater to version 1.15.1 or later. Alternatively, you may switch to a different IP provider.


