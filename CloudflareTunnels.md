---
title: Cloudflare Tunnels
description: A guide to installing Cloudflare Tunnels in TrueNAS Scale as well as on bare metal
published: true
date: 2026-02-24T14:54:47.755Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:18.633Z
---

# ![](/cloudflare.png){class="tab-icon"} What are Cloudflare Tunnels?

Cloudflare Tunnel provides you with a secure way to connect your resources to Cloudflare without a publicly routable IP address. With Tunnel, you do not send traffic to an external IP — instead, a lightweight daemon in your infrastructure (cloudflared) creates outbound-only connections to Cloudflare’s global network. Cloudflare Tunnel can connect HTTP web servers, SSH servers, remote desktops, and other protocols safely to Cloudflare. This way, your origins can serve traffic through Cloudflare without being vulnerable to attacks that bypass Cloudflare.

# 1 · Deploy Cloudflare Tunnels
# {.tabset}
## <img src="/windows-terminal.png" class="tab-icon"> Bare Metal

I don't like to run Cloudflare Tunnels in a container because it means the docker networks have to be modified in each stack to have access from the tunnel. Instead, I run the tunnel on bare metal and update it with a cronjob when necessary. This is even easier to do than a docker compose file. 

Open a free account on Cloudflare and secure a domain name. This will cost money, but not much. I pay about 66 cents per month for mine. Once you have an account and a domain name, you need to build a tunnel:

1.  Login to cloudflare and click the **Zero Trust** link in the left panel.
2.  Now go to the left panel again, click **Networks** > **Tunnels** > **\+ Create a Tunnel.**
3.  Name your tunnel then click **Save Tunnel.**
4.  At the top, you will see a section **Choose your environment**, where you need to click the corresponding blue button for the OS you are using.
5.  In the next section **Install and run a connector**, copy the command from the left box into the terminal and run it.
6.  At the bottom of the screen in the **Connectors** section, once the command has run successfully, your tunnel will appear. Click **Next**.


## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](https://wiki.hydrology.cc/screenshot_from_2023-12-11_11-42-42.png)

1. In the **Cloudflared Configuration** section, we need to add our tunnel token. 
1.  Login to cloudflare and click the **Zero Trust** link in the left panel.
1.  Now go to the left panel again, click **Networks** > **Tunnels** > **\+ Create a Tunnel.**
1.  Name your tunnel then click **Save Tunnel.**
1.  At the top, you will see a section **Choose your environment**, click the blue button for **Docker**. 
1. Below, in the **Install and run a connector** section, the tunnel token is everything after the --token line and needs to be copied to the TrueNAS install screen.

> Optionally, you can check the box in **Network Configuration** for **Host Network** if you want your tunnel to be able to access anything which is not on your TrueNAS server, like your router or other computers on your network
{.is-info}

To add apps to your tunnel, login to Cloudflare.com and click the **Zero Trust** link in the left panel > **Access** > **Tunnels**. Find the tunnel you just created and click the 3 dots at the right end of the row, and click **Configure**. Follow the Adding Endpoints section below to add apps from TrueNAS.

## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  cloudflared:
    command: tunnel --no-autoupdate run --token <token_goes_here>
    image: cloudflare/cloudflared:latest
    restart: unless-stopped
    container_name: tunnel
```

# 2 · Adding Endpoints

The next section allows you to **Add public hostnames** for your tunnel. This is where you specify what the tunnel connects to. Every service on your network you want to connect to over the internet needs to be added here. 

|  Field   |  Value   |
| --- | --- |
| **Subdomain** | this is where you would name your service |
| **Domain** | the domain name you just bought |
| **Path** | leave this blank |
| **Service Type** | usually http |
| **URL** | IP address of the service |

![](https://wiki.hydrology.cc/screenshot_from_2023-12-11_11-38-58.png)

This is an example for a bogus tunnel named “test” which points to my emby container running at 192.168.1.20 on port 9096 at a bogus domain “mydomain.com”

This example entry would create an endpoint in my existing tunnel. Now if I navigated to emby.mydomain.com in my web browser I would be routed to my Emby server in my home.

# 3 · Streaming Media

> Do not, under any circumstances, use tunnels to stream media!
{.is-danger}

It is against Cloudflare's Terms of Service to stream video through their tunnels. The better way to do this if you want to stream remotely is to use an unproxied DNS entry in Cloudflare for your domain and then port forward from your router to your container. Assuming you are using Emby, this would be a port forward on the router for port 8096 to the IP address of your server to port 8096.