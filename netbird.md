---
title: Netbird
description: A guide to installing and using Netbird
published: true
date: 2026-03-11T19:33:35.236Z
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

1. In the NetBird dashboard, navigate to **Setup Keys** ➡ **Create Setup Key**.
2. **Name:** Give it a descriptive name so you know what it was used for (e.g., `Jellyfin Server Setup`).
3. **Reusable:** 
   * Leave this **OFF (One-off)** if you are just installing NetBird on a single server. The key will automatically destroy itself after one use. 
   * Only turn this **ON** if you are deploying multiple servers simultaneously using automation (like Ansible).
4. **Expiration:** Set this to a short timeframe, such as `1 Day` or `7 Days`. 
  > **Security Warning:** Do not set the expiry to `0` (never expire). If an infinite, reusable key is ever leaked, anyone can join your private network. **Note:** Your server will *not* disconnect when this key expires! Setup keys are only used for the initial installation; after that, NetBird uses secure, auto-rotating WireGuard keys.
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
      - ./netbird-client:/etc/netbird
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
     Click **Add Target**. In the new prompt, select the NetBird Peer you set up in `1. Deploy Netbird`. Set the **Port** to the internal port your service uses (for Jellyfin, the default is `8096`). Click **Continue** and **Add Target** to return to the main setup window.

   * **Authentication:**
     If the application you are exposing already has its own secure login screen (like Jellyfin), you can skip this tab. If you are exposing a service with no built-in security, you can enforce route-level authentication here. *(See `2.3. Reverse Proxy - Authentication` for more details).*

   * **Advanced Settings:**
     It is highly recommended to enable **Pass Host Header**. This ensures that the original URL requested by the user is passed directly to your application, preventing routing errors and keeping your internal IP addresses hidden.

3. Click **Add Service** to finalize the setup. 
4. **Wait a few minutes.** NetBird needs a moment to provision the service globally and automatically generate your free SSL (HTTPS) certificate. 

> **Tip:** Once the service status shows as active, you can test it by visiting your new URL (e.g., `https://jellyfin.yourdomain.com`) from a device that is *not* connected to your local network!

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

> **💡 What does the user experience?** 
> When a user navigates to a protected URL (e.g., `https://admin.yourdomain.com`), they won't see your application right away. Instead, they are stopped by a secure NetBird interception screen asking for the required SSO login, Password, or PIN. Once they successfully authenticate, they are forwarded to your actual backend application.

# <img src="/youtube.png" class="tab-icon"> 4 · Video
https://youtu.be/skbWnMSwZcE
