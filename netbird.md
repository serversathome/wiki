---
title: Netbird
description: A guide to installing and using Netbird
published: true
date: 2026-06-29T10:43:58.651Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:37.607Z
---

# ![](/netbird.png){class="tab-icon"} What is Netbird?
NetBird combines a WireGuard®-based overlay network with Zero Trust Network Access, providing a unified open-source platform for reliable and secure connectivity

# 1 · Deploy Netbird
First, go to [https://netbird.io](https://netbird.io) and click **Get Started** to create your free account.

## 1.1 Generate a Secure Setup Key
A Setup Key acts like a temporary password that allows your server to authenticate and join your NetBird network automatically. 

1. In the NetBird dashboard, navigate to **Setup Keys > Create Setup Key**.
2. **Name:** Give it a descriptive name so you know what it was used for (e.g., `Jellyfin Server Setup`).
3. **Reusable:** 
   * Leave this **OFF (One-off)** if you are just installing NetBird on a single server. The key will automatically destroy itself after one use. 
   * Only turn this **ON** if you are deploying multiple servers simultaneously or using deployment automation (like Ansible).
4. **Expiration:** Set this to a short timeframe, such as `1 Day` or `7 Days`. 
  > **Security Warning:** Do not set the expiry to `0` (never expire). If an infinite, reusable key is ever leaked, anyone can join your private network. 
  Your server will *not* disconnect when this key expires! Setup keys are only used for the initial installation; after that, NetBird uses secure, auto-rotating WireGuard keys.
  {.is-warning}

5. Click **Create Setup Key** and copy the generated key to your clipboard.

## 1.2 Install the NetBird Agent

# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  netbird:
    cap_add:
      - NET_ADMIN
    container_name: netbird
    environment:
      - NB_SETUP_KEY=
    image: netbirdio/netbird:latest
    network_mode: host
    restart: unless-stopped
    volumes:
      - ./netbird-client:/var/lib/netbird
```
Paste your Setup Key created from above in the `- NB_SETUP_KEY=` line.

## <img src="/linux.png" class="tab-icon"> Bare Metal Linux

```bash
curl -fsSL https://pkgs.netbird.io/install.sh | sh
```
Once this is installed, execute `netbird up --setup-key <key-value>` to start the connection using the setup key created above.

## <img src="/microsoft-windows.png" class="tab-icon"> Windows
1. Download the installer from [https://pkgs.netbird.io/windows/x64](https://pkgs.netbird.io/windows/x64)
1. Click on "Connect" from the NetBird icon in your system tray
a. Alternatively you can connect through the command line by executing `netbird up --setup-key <key-value>` using the setup key created above

## Mobile

Get the app from your respective app store:
- [Android *Google Play Store*](https://play.google.com/store/apps/details?id=io.netbird.client)
- [iOS *Apple App Store*](https://apps.apple.com/us/app/netbird-p2p-vpn/id6469329339)
{.links-list}

# 2 · Reverse Proxy
NetBird's built-in reverse proxy lets you expose local services over HTTPS without opening a single port on your router. Every service you publish lives on a **cluster** — a group of one or more proxy instances that serve a single apex domain. There are two kinds:

- **Shared cluster** — the proxy is run by NetBird (on NetBird Cloud) or by whoever operates your management server. Zero setup: you just point a CNAME at their infrastructure. **This is what Sections 2.1 and 2.2 below use.**
- **Account cluster** — a proxy *you* run on your *own* box, bound exclusively to your account, with traffic terminating on hardware you control. This is the **Bring Your Own Proxy (BYOP)** feature covered in [Section 2.4](#h-24-clusters-bring-your-own-proxy).

Start with a shared cluster to get going in minutes, then graduate to your own cluster when you want traffic to stay on your own server.

## 2.1 Custom Domains
By default, NetBird assigns your proxy services to a provided cloud domain. However, if you own your own domain (e.g., `yourdomain.com`), you can attach it to NetBird to make your services look cleaner and easier to remember.

1. **Add the Domain to NetBird:**
   * In the NetBird dashboard, navigate to the **Reverse Proxy > Custom Domains** tab.
   * Click **Add Domain** and enter your root domain (e.g., `yourdomain.com`).
   * NetBird will generate specific **DNS records** (usually a CNAME record) that you need to add to your domain registrar. Leave this window open.

2. **Configure your DNS Provider:**
   * Open a new tab and log into the service where you manage your domain's DNS (e.g., Cloudflare, Namecheap, GoDaddy).
   * Navigate to your domain's **DNS Management** or **Records** page.
   * Add the records exactly as NetBird requested:
     * **CNAME Record:** This physically routes traffic from your domain to NetBird's cloud infrastructure. You will likely point a wildcard (`*`) or specific subdomain to NetBird's provided target address.

  > **Important note for Cloudflare users:** If you are using Cloudflare for DNS, make sure the **Proxy status** (the orange cloud icon) is toggled **OFF** (set to "DNS Only") for these specific CNAME records. Because NetBird is now acting as your Reverse Proxy, leaving Cloudflare's proxy enabled will cause conflicts and prevent NetBird from verifying your domain or issuing SSL certificates.
  {.is-warning}

3. **Verify and Wait:**
   * Head back to the NetBird dashboard and click **Verify**. 
   * **Wait for propagation:** DNS changes can take anywhere from 5 minutes to a few hours to propagate globally. Once verified, NetBird will status will update to "Active" and automatically secure your domain with a free SSL certificate.

Once your custom domain is active, it will be available in the dropdown menu when creating new routes in the **Services** tab.

## 2.2 Services
Once your server is connected to the NetBird mesh, you can expose its local applications to the internet without opening any ports on your router.

1. In the NetBird dashboard, navigate to the **Reverse Proxy > Services** tab.
2. Click the **Add Service** button. This will open a configuration window with several steps:

   * **Details:** 
     Enter a memorable subdomain for your service (e.g., `jellyfin` or `media`) and select the domain you wish to use.
   
   * **Targets:**
     Click **Add Target**. In the new prompt, select the NetBird Peer you set up in [1. Deploy Netbird](https://wiki.serversatho.me/en/netbird#h-1-deploy-netbird) Set the **Port** to the internal port your service uses (for Jellyfin, the default is `8096`). Click **Continue** and **Add Target** to return to the main setup window.

   * **Authentication:**
     If the application you are exposing already has its own secure login screen (like Jellyfin), you can skip this tab. If you are exposing a service with no built-in security, you can enforce route-level authentication here. *(See [2.3. Reverse Proxy - Authentication](https://wiki.serversatho.me/en/netbird#h-23-authentication) for more details).*

   * **Advanced Settings:**
     It is highly recommended to enable **Pass Host Header**. This ensures that the original URL requested by the user is passed directly to your application, preventing routing errors and keeping your internal IP addresses hidden.

3. Click **Add Service** to finalize the setup. 
4. **Wait a few minutes.** NetBird needs a moment to provision the service globally and automatically generate your free SSL (HTTPS) certificate. 
> 
> **Tip:** Once the service status shows as active, you can test it by visiting your new URL (e.g., `https://jellyfin.yourdomain.com`) from a device that is *not* connected to your local network!
{.is-info}


## 2.3 Authentication
While applications like Jellyfin manage their own users and login screens (meaning you skip this step), many self-hosted tools (like network dashboards, basic web servers, or admin panels) lack strong built-in security. NetBird allows you to put an identity layer in front of these routes, forcing visitors to authenticate before they can even reach the application.

1. **Access the Authentication Settings:**
   * You can configure this during the initial `Add Service` wizard, or by clicking **Edit** on an existing route in the **Reverse Proxy > Services** tab.
   * Navigate to the **Authentication** tab.

2. **Choose an Authentication Method:**
   * By default, if you don't add any authentication, the route remains **Public** (traffic flows directly to your application). 
   * To protect the route, click the **⊕ Add** button next to your preferred security method:
     * **SSO (Single Sign-On):** Visitors must log in using the Identity Provider tied to your NetBird network (e.g., Google, Microsoft, OIDC). Best for strictly limiting access to known users in your NetBird environment.
     * **Password:** Visitors will be prompted to enter a specific password you define. Great for quickly protecting an app or sharing it securely without requiring the visitor to have an actual account.
     * **PIN Code:** Visitors must enter a numeric PIN code to access the service. Useful for quick access on mobile devices or TV interfaces.

3. **Save and Apply:**
   * Once you've configured your desired method, click the orange **Save Changes** button. The new security rules will apply to your proxy route within minutes.
> 
> **💡 What does the user experience?** 
> When a user navigates to a protected URL (e.g., `https://admin.yourdomain.com`), they won't see your application right away. Instead, they are stopped by a secure NetBird interception screen asking for the required SSO login, Password, or PIN. Once they successfully authenticate, they are forwarded to your actual backend application.
{.is-success}

## 2.4 Clusters (Bring Your Own Proxy)
Everything above runs on a **shared cluster** — NetBird's own proxy infrastructure handles TLS and routes your traffic. That's the easy path, but it means your traffic terminates on someone else's servers.

**Bring Your Own Proxy (BYOP)** lets your account run its own reverse proxy on your own box. The proxy connects to NetBird's management service like any other proxy, but it's bound to a single account: only *your* services route through it, and the apex domain you choose is reserved across the management instance. For a homelabber, this is the sweet spot — you keep NetBird's slick dashboard and mesh routing, but the proxy and the Let's Encrypt certs live on a server *you* control (a VPS, for example).

You'd reach for BYOP when you want:
- **Full control over traffic** — proxied traffic terminates on infrastructure you operate, not on shared clusters.
- **Specific geographic placement** — pick the region/provider you need.
- **Your own TLS and domain** — the proxy issues certificates directly via Let's Encrypt for a wildcard under a domain you own.

In the dashboard, BYOP proxies live under **Reverse Proxy > Clusters** and show up as **account clusters**.

### 2.4.1 Shared vs. Account Clusters
Both cluster types appear together on the **Clusters** page. The **Type** badge marks each row as shared or account, the **Status** column shows whether at least one proxy has heartbeated in the last two minutes, and the **Features** column lists what the connected proxies support.

| | **Shared cluster** | **Account cluster** (BYOP) |
|---|---|---|
| Who runs it | NetBird / your management operator | You |
| Who can use it | Every account on the management instance | Only your account |
| Apex domain | Provided by the platform | Provided by you (you own the DNS) |
| TLS | Managed by the platform | Issued by the proxy you run (ACME or your own certs) |
| Registration token | Management-wide | Account-scoped (one per account) |
| Geographic placement | Wherever the platform runs proxies | Wherever you run the container |
| Delete from dashboard | Not allowed | Allowed (account owner only) |


There's **no functional difference at the data plane** — a service behaves identically once a request lands on either cluster type. The choice is purely operational: shared = zero effort, account = control over location, TLS, and the data path.

The rest of this section sets up an **account cluster**. If you just want a shared cluster, there's nothing to do here — pick the platform-provided domain back in [Section 2.2](#h-22-services).


### 2.4.2 Setup Walkthrough
The dashboard provides a wizard that generates the proxy token, shows the DNS records to add, and emits a ready-to-run command.

1. **Open the wizard.** Navigate to **Reverse Proxy > Clusters**. The table lists every cluster your account can reach. Click **Setup Self-Hosted Cluster**.

2. **Choose your domain.** In the **Domain** tab, enter the domain this cluster will be reachable on, e.g., `proxy.yourdomain.com`. This becomes the proxy's `NB_PROXY_DOMAIN` and the suffix of every service URL hosted on it (`{subdomain}.proxy.yourdomain.com`). Click **Continue**.

3. **Configure DNS records.** In the **DNS Records** tab, add these two records at your registrar:

   | Type  | Name                    | Content                  |
   |-------|-------------------------|--------------------------|
   | A     | `proxy.yourdomain.com`  | Your server's public IP  |
   | CNAME | `*.proxy.yourdomain.com`| `proxy.yourdomain.com`   |
   {.dense}

   The wildcard record is required so every service domain (`{subdomain}.proxy.yourdomain.com`) resolves to your proxy. If you're on Cloudflare, set these records to **DNS Only** (grey cloud). Click **Continue**.

4. **Run the proxy.** The **Run the Proxy** tab generates a **one-time, account-scoped token** and embeds it into a command. The wizard hands you a `docker run`, but here's the equivalent Dockge-friendly compose stack to run on your server:

   ```yaml
   services:
     netbird-proxy:
       container_name: netbird-proxy
       image: netbirdio/reverse-proxy:latest
       restart: unless-stopped
       environment:
         - NB_PROXY_CERTIFICATE_DIRECTORY=/certs
         - NB_PROXY_MANAGEMENT_ADDRESS=https://api.netbird.io
         - NB_PROXY_ACME_CERTIFICATES=true
         - NB_PROXY_DOMAIN=proxy.yourdomain.com
         - NB_PROXY_LOG_LEVEL=info
         - NB_PROXY_TOKEN=
       ports:
         - "443:443"
         - "80:80"
       volumes:
         - ./netbird-certs:/certs
   ```

   Paste the generated token into the `NB_PROXY_TOKEN=` line and set `NB_PROXY_DOMAIN` to the domain from Step 2. On NetBird Cloud, leave `NB_PROXY_MANAGEMENT_ADDRESS` as `https://api.netbird.io`; on a self-hosted management server, use your own management URL.

   > **Copy the token now.** The plain token is shown **only once**, at the moment it's generated. Store it somewhere safe before closing the modal. If you lose it, revoke it and generate a new one.
   {.is-danger}


    > With the default `tls-alpn-01` challenge you only need port 443 — you can drop the `80:80` mapping. Keep port 80 only if you switch to the `http-01` challenge.
    {.is-info}




### 2.4.3 Using Your Cluster for Services
Once the cluster is connected, your BYOP domain shows up as a normal **Cluster** option in the service-creation flow:

1. Go to **Reverse Proxy > Services > Add Service**.
2. Choose a subdomain and pick your BYOP domain (`proxy.yourdomain.com`) as the base domain.
3. Configure targets, authentication, and access restrictions exactly as in [Section 2.2](#h-22-services) and [Section 2.3](#h-23-authentication).

Traffic to `subdomain.proxy.yourdomain.com` is now received by *your* proxy, terminated locally with a Let's Encrypt certificate, and forwarded over WireGuard to the target peer.


# <img src="/youtube.png" class="tab-icon"> 3 · Video
https://youtu.be/-yfE3Lb3hTI
https://youtu.be/skbWnMSwZcE



# <img src="/eagle.png" class="tab-icon"> 4 · Would You Like to Know More?
https://blog.serversatho.me/best-vpn-ever/