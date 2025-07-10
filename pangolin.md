---
title: Pangolin
description: A guide to installing Pangolin
published: true
date: 2025-07-10T19:53:51.538Z
tags: 
editor: markdown
dateCreated: 2025-06-13T13:04:34.352Z
---

# ![](/pangolin.png){class="tab-icon"} What is Pangolin?
Pangolin at its core is a self-hosted tunneled reverse proxy with identity and access management, designed to securely expose private resources through encrypted WireGuard tunnels running in user space. Think self hosted Cloudflare tunnels.
> 
> Read the [official documentation](https://docs.fossorial.io/Getting%20Started/overview)
{.is-success}


# 1 · Prerequisites
- A Linux system with root access and a public IP address *(we recommend Ubuntu or Debian based systems)*
- A domain name pointed to your server's IP address
- TCP ports 80, 443, and UDP port 51820 exposed to your Linux instance.

# 2 · Choosing a VPS
Pangolin is best run from somewhere outside your network, ideally in the cloud. As such, you need to have a VPS to install Pangolin.

A minimal VPS instance with 1 vCPU, 1GB RAM, and 8GB SSD will perform perfectly well for most use cases. In some cases, you may be able to get away with even less.

The Pangolin docs recommend [this option from Rack Nerd](https://my.racknerd.com/index.php?rp=/store/kvm-vps-latest-special-promos) and honestly it's a great choice, but any VPS will do.

# 3 · Installation
Pangolin is installed on bare metal using this command:
```bash
wget -O installer "https://github.com/fosrl/pangolin/releases/download/1.5.1/installer_linux_$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')" && chmod +x ./installer
```

The downloaded files will be named `installer` in the current directory.

The installer must be run as root. If you're not already root, switch to the root user or use sudo:
```bash
sudo ./installer
```

# 4 · Installer Configuration
The installer will prompt you for the following basic information. For example:

1. **Base Domain Name:** Enter your base fully qualified domain name (without any subdomains) Example: example.com
1. **Dashboard Domain Name:** The domain where the application will be hosted. This is used for many things, including generating links. You can run Pangolin on a subdomain or root domain. Example: `pangolin.example.com`
1. **Let's Encrypt Email:** Provide an email address for SSL certificate registration with Lets Encrypt. This should be an email you have access to.
1. **Tunneling:** You can choose not to install Gerbil for tunneling support - in this config it will just be a normal reverse proxy. See how to [use without tunneling](https://docs.fossorial.io/Pangolin/without-tunneling).

## 4.1 Admin User Setup

You'll need to configure the admin user. This is the first user in the system. You will log in initially with this user.

1. **Admin Email:** Defaults to `admin@yourdomain.com` but can be customized
1. **Admin Password:** Must meet these requirements:
    - At least 8 characters
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one digit
    - At least one special character

## 4.2 Security Settings

It will ask you to configure some basic security options. For example:

1. **Signup Without Invite:** Choose whether to disable user registration without invites (defaults to disabled). This removes the "Sign Up" button on the login form and is recommended for private deployments.
1. **Organization Creation:** Decide if users can create their own organizations (defaults to enabled)

## 4.3 Email Configuration

Decide whether to enable email functionality. This allows Pangolin to send transactional emails like OTP or email verification requests.

If enabled, you'll need to provide:
- SMTP host
- SMTP port (defaults to 587)
- SMTP username
- SMTP password
- No-reply email address. This is the sender email address that Pangolin will email from. Many times this should be the same as the username.

# 5 · Pangolin Configuration

## 5.1 Create a Org

An org is a way to collect sites, users, and resources.

When you log into the app for the first time you will be prompted to create an org. Simply choose a name and an ID. Note that the ID can not be changed later!

## 5.2 Create a site

A site is a remote location that you want to proxy through the tunnel and system. For example your home server, or a IOT device. A site will terminate one tunnel.

1. Head to the Sites tab and select the Add Site button (or use the tab in the setup workflow)
1. Give your site a name like "HomeLab"
1. Choose your **TunnelType**. You can either use the Newt client (recommended) or a standard WireGuard tunnel.
1. Check the circle **I have copied the config**
1. Copy the Newt command for the Operating System you have chosen
1. Press **Create Site**

## 5.3 Connect a Tunnel

1. Assuming you chose Newt above, deploy the docker compose file or bash command to your endpoint
1. Click **Save General Settings**
1. In the left pane, click **Sites** and you should see you new site as **Online**. If not, give it a minute and refresh the page.


## 5.4 Create a Resource

1. Head to the Resources tab and select the **Add Resource button** (or use the tab in the setup workflow)
1. Give your resource a name like "Bitwarden"
1. Choose a subdomain for this resource. The subdomain must be **globally unique** across all orgs and sites
1. Choose the site that this resource is at. The resource target must be accessible behind the tunnel attached to this site.
1. Press **Create Resource**

## 5.5 Add Targets and Authentication
### 5.5.1 Target

1. You should now be on the Connectivity page under your new resource
1. If you would like to secure this site with https, leave the **Enable SSL** toggle enabled
1. Add a target for this resource. If your resource is accessible on your internal network at `http://192.168.1.24:8080` for example, then choose the following:
	a. **Method:** `HTTP` 
  b. **IP Address:** `192.168.1.24` 
  c. **Port:** `8080`
1. Press **Add Target** and you will see the target added to the list and enabled.
1. Press **Save Target**
1. Try to access your resource by clicking the url at the top


> After you create your resource if you are using https certificates with Let's Encrypt (default) then you must wait some time after a target is created for your certificate to be granted and loaded by Traefik. This should take no more than a few minutes. For instant access, consider setting up wildcard certificates.
{.is-info}

### 5.5.2 Authentication

1. Choose the Authentication page under the resource

By default the resource is protected with your same Pangolin account. When opening the resource it just loads because you are already logged in. If you were not, you would first be redirected to Pangolin to login before being sent back to the resource.

If you would like to disable Pangolin auth, you can disable the `Use Platform SSO` toggle.

> It is not recommended to expose a resource without some form of authentication. Only do this if you need to for the functionality of the resource or you trust the built-in auth.
{.is-warning}


## 5.6 Invite Users (optional)

1. Head to the Users and Roles tab
1. Press **Invite User**
1. Enter an email for the new user. If you have setup SMTP during the setup you can choose to send an email invite to the new user
1. Select the role for the new user. All users must have a role. The admin role gives the user access to all resources and to create new resources and sites. The member role only provides access to resources explicitly attached to the role (none by default).
1. Choose how long this invite will be valid for and choose **Create Invitation**
1. If you chose not to send the email or it is not setup, then be sure to copy the invite and send it to the user

The new user will be prompted to setup a password and verify their email (if SMTP is supported). They will show up in your table once they confirm their account.



# <img src="/youtube.png" class="tab-icon"> 6 · Video

[](https://youtu.be/1fKqQi-VuNM)
