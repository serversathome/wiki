---
title: qBittorrent
description: A guide to installing qBittorrent through docker via compose
published: true
date: 2025-09-22T15:45:50.552Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:36:26.298Z
---

# ![](/qbittorrent.png){class="tab-icon"} What is qBittorrent?
qBittorrent is a free and open-source software that aims to provide the same features as µTorrent, such as polished user interface, no ads, search engine, torrent creation tool and more. It runs on all major platforms (Windows, Linux, macOS, FreeBSD, OS/2) and supports many Bittorrent extensions.

# 1 · Deploy qBittorrent
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Hotio + VPN

```yaml
services:
  qbittorrent:
    container_name: qbittorrent
    image: ghcr.io/hotio/qbittorrent
    restart: unless-stopped
    ports:
      - 8080:8080
    environment:
      - PUID=568
      - PGID=568
      - UMASK=002
      - TZ=America/New_York
      - WEBUI_PORTS=8080/tcp,8080/udp
      - VPN_ENABLED=true
      - VPN_CONF=wg0
      - VPN_PROVIDER=generic
      - VPN_LAN_NETWORK=10.99.0.0/24
      - VPN_LAN_LEAK_ENABLED=false
      - VPN_EXPOSE_PORTS_ON_LAN=
      - VPN_AUTO_PORT_FORWARD=true
      - VPN_AUTO_PORT_FORWARD_TO_PORTS=
      - VPN_FIREWALL_TYPE=auto
      - VPN_HEALTHCHECK_ENABLED=false
      - VPN_NAMESERVERS=
      - PRIVOXY_ENABLED=false
    cap_add:
      - NET_ADMIN
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=1
    volumes:
      - /mnt/tank/configs/qbittorrent:/config
      - /mnt/tank/media:/media
```

This qBittorrent container is from hotio and uses a Wireguard VPN to protect traffic. 

<details><summary><strong>Environment Variables Explanations</strong> (click to expand)</summary>

|  Variable   | Value    |
| --- | --- |
| `VPN_CONF` | With VPN\_CONF you can set the name used for your WireGuard config. There needs to be a file `wg0.conf` located in `/config/wireguard` for the VPN to start. |
| `VPN_LAN_NETWORK` | The environment variable VPN\_LAN\_NETWORK can be set to for example 192.168.1.0/24, 192.168.1.0/24,192.168.44.0/24, so you can get access to the webui or other additional ports. If for example you were to pick 192.168.0.0/24, every device with an ip in the range 192.168.0.0 - 192.168.0.255 on your LAN is allowed access to the webui. |
| `VPN_EXPOSE_PORTS_ON_LAN` | If you need to expose ports on your LAN you can use VPN\_EXPOSE\_PORTS\_ON\_LAN. For example VPN\_EXPOSE\_PORTS\_ON\_LAN=7878/tcp,9117/tcp, will block those ports on the vpn interface, so that there's no risk that they might be exposed to the world and allow access to them from your LAN. Some images also have a WEBUI\_PORTS environment variable that does basically the same for the vpn part. For those apps that support it, it'll also change the port on which the app runs. |
| `VPN_AUTO_PORT_FORWARD` | Auto retrieve a forwarded port and configure the supported app if set to true or if you can manually request/set a forwarded port in the VPN provider's web interface, fill in the port number (just the number). |
| `VPN_AUTO_PORT_FORWARD_TO_PORTS` | Adds a redirect for the forwarded port from your vpn provider to the internal port on which the app runs, ports in this list are also not blocked on the wireguard interface, so this var is also useful if you want to expose a port on both your LAN and VPN. Values like 32400/tcp will use the port from VPN\_AUTO\_PORT\_FORWARD to create the redirect or if set to true the forwarded port from pia/proton. Use 3000@3001/tcp,3002@3003/tcp syntax for extra static redirects. The only known usecase as of right now is Plex and exposing it on the VPN with a non configurable forwarded port, because it's not possible to run Plex on anything else but 32400. |
| `VPN_FIREWALL_TYPE` | Possible values are auto, legacy or nftables. The default is auto, this will try to use the most modern method available. If this doesn't work, you can try forcing it to legacy or nftables. |
| `VPN_HEALTHCHECK_ENABLED` | This is almost never needed, only in very rare cases (mostly when using PIA). |
| `VPN_NAMESERVERS` | Possible values are `wg`, `8.8.8.8` or `1.1.1.1@853#cloudflare-dns.com` seperated by a `,`. The value `wg` will use the nameservers from the `wg0.conf` file. The value `8.8.8.8` is to use a plain old nameserver. The value `1.1.1.1@853#cloudflare-dns.com` will add a DNS over TLS nameserver, this will override all other regular nameservers. Leaving the variable empty will allow Unbound to work in recursive mode.   |

> For more info on these values, look [here](https://hotio.dev/containers/qbittorrent/#__tabbed_5_2) 
{.is-info}

</details>
  
> When you start this container it will fail until you add the VPN config file. See the [Example Wireguard](https://wiki.serversatho.me/en/qBittorrent#example-wireguard-wg0conf-file) section below
{.is-warning}



## <img src="/docker.png" class="tab-icon"> Gluetun + qBit

> There have been widespread reports of issues with this container. The main one being it will work for awhile then go down, or the port will be open then close on its own after 10 minutes or less. As such, I am not recommending this for the time being. Use at your own risk - no support will be answered for issues with this container
{.is-danger}

> In Discord many people have said this container fixes the Gluetun issue. I have not tested it, but if you are using Gluetun and you get randomly discconnected every so often, try this method: [qSticky](https://github.com/monstermuffin/qSticky).
{.is-info}


```yaml
services:
  gluetun:
    cap_add:
      - NET_ADMIN
    container_name: gluetun
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - TZ=America/New_York
      - VPN_SERVICE_PROVIDER=custom
      - VPN_TYPE=wireguard
      - WIREGUARD_ENDPOINT_IP=
      - WIREGUARD_ENDPOINT_PORT=
      - WIREGUARD_PUBLIC_KEY=
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_PRESHARED_KEY=
      - WIREGUARD_ADDRESSES=
      - FIREWALL_VPN_INPUT_PORTS=6881
      - SERVER_COUNTRIES= #optional
      - DNS_ADDRESS= #optional
    image: qmcgaw/gluetun:latest
    ports:
      - '8080:8080'
      - 6881:6881/udp
      - 6881:6881/tcp
    restart: unless-stopped

  qbittorrent:
    container_name: qbittorrent
    depends_on:
      - gluetun
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - WEBUI_PORT=8080
    image: linuxserver/qbittorrent:latest
    network_mode: service:gluetun
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/qbittorrent:/config
      - /mnt/tank/media:/media
```

| Variable    |  Value   |
| --- | --- |
| `VPN_SERVICE_PROVIDER` | This is setup as *custom* to accommodate any provider but there are many more [providers](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers). |
| `FIREWALL_VPN_INPUT_PORTS` | Comma separated list of ports |
| `SERVER_COUNTRIES` | Comma separated list of countries |

To add port forwarding, on the `FIREWALL_VPN_INPUT_PORTS` add a comma with no space and then the port which you want to forward. Make sure to make the change in the qbittorrent connection settings as well as shown below.

> For more info on this container, look [here](https://github.com/qdm12/gluetun-wiki?tab=readme-ov-file)
{.is-info}



## <img src="/docker.png" class="tab-icon"> NordVPN + qBit


```yaml
services:
  nordvpn:
    image: ghcr.io/bubuntux/nordlynx
    container_name: nordvpn
    cap_add:
      - NET_ADMIN # Required for VPN
      - NET_RAW
      - SYS_MODULE
    environment:
      - PRIVATE_KEY={ENTER YOUR PRIVATE KEY HERE}
      - CONNECT=United_States # Set preferred country/server
      - TECHNOLOGY=NordLynx # Or OPENVPN
      - NETWORK={ENTER YOUR LAN SUBNET HERE;i.e. 192.168.1.0/24} # LAN subnet allowed to access UI
    ports:
      - 8081:8080 #expose port for qbittorrent webgui. Must expose for the VPN container as Qbit passes all traffic through this container.
    dns: #This helps if you are getting DNS errors when deploying the container.
      - 1.1.1.1
      - 8.8.8.8
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=1 #This disables the ipv6 table otherwise you'll get ipv6 errors.
      - net.ipv4.ip_forward=1
    volumes:
      - /mnt/tank/configs/nordvpn:/config
    devices:
      - /dev/net/tun
    restart: unless-stopped
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    network_mode: container:nordvpn # Force traffic through VPN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
      - WEBUI_PORT=8080
    volumes:
      - /mnt/tank/configs/qbittorrent:/config
      - /mnt/tank/media:/media  # Can set as /downloads if that is how your file structure is designed.
    depends_on:
      - nordvpn
    restart: unless-stopped
networks:
  default:
    driver: bridge
```

To get your Private Key needed to make this container work, you will need to log into your NordVPN account and head to the "NordVPN" service located on the left menu option.  Scroll down until you see the "Access Token" option and click on "Get Access Token".  You will likely need to verify your email address.  Once verified, click on the "Generate a New Token" option and leave set to 30 day period, as the token will not be necessary after being used to generate your Private Key.

You will now use the Token you have generated to generate your Private Key by using the following command to run a container in a removal state, so that it will display in the terminal what your Private Key is for NordVPN to be able to enter into the compose file above.  You will want to use a Shell that has access to the Docker service.  If you have been using Dockge installed on a Truenas instance, you can shell into the Dockge app to use this command to generate your Private Key.

```bash
docker run --rm --cap-add=NET_ADMIN -e TOKEN={{{TOKEN}}} ghcr.io/bubuntux/nordvpn:get_private_key
```

>For more info on this container, look [here](https://github.com/bubuntux/nordlynx)
{.is-info}

# 2 · Example Wireguard wg0.conf File

This is an example of how your `wg0.conf` file should look like. If there's a lot of extra stuff, remove it unless you know what it's there for.

```yaml
[Interface]
Address = 10.171.142.169
PrivateKey = supersecretkey
MTU = 1320
DNS = 10.128.0.1

[Peer]
PublicKey = supersecretkey
PresharedKey = supersecretkey
Endpoint = 1.2.3.4:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 15
```

> For more clarification on these values, [look here](https://hotio.dev/containers/qbittorrent/#__tabbed_2_1)
{.is-info}

> If you are using the hotio container for qBit with a VPN this file needs to be added to `/config/wireguard` (in the folder holding the qBit configuration files) before the container can run
{.is-warning}


# 3 · Testing Open Ports

To test if your port forward is working correctly, shell into qBit and execute:
```bash
curl ip.me
```
to get the IP address you are using. Then plug that IP + your port into [this tool](https://www.yougetsignal.com/tools/open-ports/).

# 4 · Testing Functionality

To test everything is working, try downloading [this Ubuntu torrent](https://releases.ubuntu.com/24.04/ubuntu-24.04.2-live-server-amd64.iso.torrent). Once you have it downloaded, upload it into qBit by using the ➕ button in the upper left corner.

A result of `downloading` at *any* speed indicates a success. If it stalls or errors something is wrong with permissions or dataset structure.

# <img src="/youtube.png" class="tab-icon"> 5 · YouTube Walkthrough

[https://www.youtube.com/watch?v=WVM3Wgb290g](https://youtu.be/I4SRwmKLfQQ)

# 6 · qBittorrent Configuration

## 6.1 Logging In

To login to the webUI, navigate to http://{serverIP}:8080. If when you first try to open the app and you receive a blank page which just says “Unauthorized”, don't worry, you did everything correctly. Click the URL bar and hit <kbd>Enter</kbd> and you should be forwarded to the login screen. The default username is *admin* and the password is set randomly. To see it, go into the logs and look for this line:

![](/screenshot_from_2024-08-26_07-55-35.png)

> If you are having an issue where qBit is running but you cannot reach the webUI and you are coming into your network over VPN, be sure to add the VPNs IPv4 range to the `VPN_LAN_NETWORK` line in a comma separated list, no spaces
{.is-info}


## 6.2 Configuration Options

Click on the cog icon :gear: in the bar or select **Tools** > **Options** in the tool bar. I will tell you what to change tab by tab. If it is not mentioned, it is left as the default value

# {.tabset}
## Downloads

The **Default Save Path** needs to be modified to be **/media/downloads** if you followed the [Folder Structure](/Folder-Structure) guide. 

![screenshot_from_2025-05-07_08-20-01.png](/screenshot_from_2025-05-07_08-20-01.png)

## Connection

In the **Listenting Port** section, add the port given to you by AirVPN (see the [AirVPN page](/AirVPN) for instructions).

Uncheck all the boxes under **Connection Limits**.

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_14-39-29.png)

## Bittorrent

If you are using private trackers only, uncheck all the boxes in the **Privacy** grouping. If you are using public trackers, leave this section default.

Uncheck the box for **Torrent Queuing**. 

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_14-39-42.png)

## Web UI

### VueTorrent

To get VueTorrent (I put mine in `/mnt/tank/media/downloads/VueTorrent`) run this command in the TrueNAS shell:
```bash
git clone --single-branch --branch latest-release https://github.com/VueTorrent/VueTorrent.git /mnt/tank/media/downloads/
```

In the Web UI tab, check the box for **Use Alternative WebUI** and enter the path of the VueTorrent folder (`/media/downloads/VueTorrent`). 

![](/screenshot_from_2024-12-14_15-09-59.png)

I keep mine up to date (assuming I save it in `/mnt/tank/media/downloads`) by adding this command to TrueNAS in **System** → **Advanced Settings** → **Cron Jobs** running as `root`:
```bash
git -C /mnt/tank/media/downloads/VueTorrent pull
```

### Qui
To use Qui instead, visit the [Qui page](/qui) to deploy the container alongside qBittorrent.

## Advanced

Change the **Network Interface** to **VPN** (yours might also say **wg0** or **tun0**).

![](https://wiki.hydrology.cc/screenshot_from_2023-12-14_14-40-12.png)

# <img src="/youtube.png" class="tab-icon"> 7 · Troubleshooting Video

[https://youtu.be/v3cv-LO9Ufo](https://youtu.be/v3cv-LO9Ufo)
