---
title: Forgejo
description: A guide to deploying Forgejo
published: true
date: 2026-09-02T10:39:53.290Z
tags: 
editor: markdown
dateCreated: 2026-08-25T18:25:30.503Z
---

# <img src="/forgejo.png" class="tab-icon"> What is Forgejo?

**Forgejo** is a self-hosted software forge. In plain terms, it is your own GitHub: git hosting over HTTP and SSH, issues, pull requests, wikis, releases, organizations, CI/CD, and a built-in container registry, all from a single lightweight Go binary.

# 1 · Deploy Forgejo

# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  forgejo:
    image: codeberg.org/forgejo/forgejo:16
    container_name: forgejo
    depends_on:
      - forgejo-db
    environment:
      - USER_UID=568
      - USER_GID=568
      - FORGEJO__server__DOMAIN=git.serversatho.me
      - FORGEJO__server__ROOT_URL=https://git.serversatho.me/
      - FORGEJO__server__HTTP_PORT=3000
      - FORGEJO__server__DISABLE_SSH=true
      - FORGEJO__server__LFS_START_SERVER=true
      - FORGEJO__database__DB_TYPE=postgres
      - FORGEJO__database__HOST=forgejo-db:5432
      - FORGEJO__database__NAME=forgejo
      - FORGEJO__database__USER=forgejo
      - FORGEJO__database__PASSWD=CHANGE_ME_DB
      - FORGEJO__service__DISABLE_REGISTRATION=false
      - FORGEJO__service__ALLOW_ONLY_EXTERNAL_REGISTRATION=true
      - FORGEJO__repository__DEFAULT_BRANCH=main
      - FORGEJO__cron.update_mirrors__SCHEDULE=@every 30m
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/forgejo/data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro

  forgejo-db:
    image: postgres:17
    container_name: forgejo-db
    environment:
      - POSTGRES_USER=forgejo
      - POSTGRES_PASSWORD=CHANGE_ME_DB
      - POSTGRES_DB=forgejo
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/forgejo/db:/var/lib/postgresql/data
```

1. Create the directories: `mkdir -p /mnt/tank/configs/forgejo/{data,db}`
2. Change `CHANGE_ME_DB` in **both** services to the same strong password
3. Replace `git.serversatho.me` with your own hostname everywhere it appears



## <img src="/truenas.png" class="tab-icon"> TrueNAS

Forgejo is in the **Community** train of the TrueNAS Apps catalog.

1. Navigate to **Apps** in the TrueNAS UI
2. Search for **Forgejo** and click **Install**
3. Configure the following:
   - **Host Path** for data: `/mnt/tank/configs/forgejo/data`
   - **Host Path** for Postgres: `/mnt/tank/configs/forgejo/db`
   - **Web Port**: `3000`
   - **Root URL**: `https://git.serversatho.me/`
4. Click **Save**
5. Under the app's config, disable SSH if the option is offered

# 2 · First Run

## 2.1 Complete the installer

Browse to `http://<truenas-ip>:3000`. Most fields are pre-filled from your environment variables. Confirm the database settings and scroll to **Administrator Account Settings**.

Create your admin account here, as a local username and password account.

> **Do not make your admin account an OAuth account.**
> Section 5 adds "Sign in with GitHub" so contributors do not need to register. That OAuth application is owned by your GitHub account. If GitHub ever suspends you, which is the exact scenario this build exists to survive, the OAuth app dies with it and nobody can log in to your forge, including you. A local admin account is your way back in.
{.is-danger}

## 2.2 Lock the installer

Once setup completes, confirm `INSTALL_LOCK = true` in `/mnt/tank/configs/forgejo/data/gitea/conf/app.ini`. The installer should never be reachable again.

## 2.3 HTTPS is the only clone method

This build turns SSH off. Every repository offers one clone URL:

```bash
git clone https://git.serversatho.me/yourname/yourrepo.git
```

Authenticate with a **Personal Access Token** (Settings → Applications) used as the password. Run `git config --global credential.helper store` so you are not pasting it on every push.

There are three reasons this is the right default rather than a compromise:

- **A reverse proxy cannot route SSH.** It routes by hostname, read from the HTTP `Host` header or the TLS SNI, and SSH has neither. Offering SSH publicly means a second port forward straight to your NAS on top of the one NPM already uses.
- **Your contributors have no SSH key here anyway.** They sign in through GitHub OAuth, so they never uploaded one. HTTPS with a token is their only working option regardless of what you configure.
- **Port 22 is already TrueNAS's.** Keeping SSH means running Forgejo on an alternate port and making sure `SSH_PORT` matches, which is one more thing to get wrong for a feature nobody is using.



# 3 · Exposing Forgejo

This guide assumes **Nginx Proxy Manager** with ports 80 and 443 forwarded to it. If you would rather not open ports at all, skip to 3.3.

## 3.1 Nginx Proxy Manager

Create a new Proxy Host:

| Setting | Value |
|---------|-------|
| Domain Names | `git.serversatho.me` |
| Scheme | `http` |
| Forward Hostname / IP | TrueNAS LAN IP |
| Forward Port | `3000` |
| Block Common Exploits | On |
| Websockets Support | **On** |
| SSL | Request a new Let's Encrypt certificate |
| Force SSL | On |
| HTTP/2 Support | On |

Under the **Advanced** tab, add:

```nginx
client_max_body_size 0;
proxy_request_buffering off;
```

> **Do not remove the Advanced block.** `client_max_body_size 0` lifts NPM's default upload cap. Without it, pushing a container image or a large release asset fails with a `413` that the docker client reports as the unhelpful `error from registry: unknown`. `proxy_request_buffering off` stops NPM writing multi-hundred-megabyte layers to disk before forwarding them.
{.is-danger}



## 3.2 Certificates

Let's Encrypt HTTP-01 works if port 80 reaches NPM from the internet. Some residential ISPs block inbound 80. If your cert request fails, switch NPM to a **DNS challenge** using an API token from your DNS provider, which does not need any inbound port.

You also need **NAT hairpin** (also called NAT loopback) on your router so machines *inside* your LAN can reach `git.serversatho.me`. UniFi does this by default. If your forge works from your phone on cell data but times out from your desk, hairpin is the reason.

## 3.3 Without opening ports

Both of these publish to the internet without a single inbound port on your home connection, by having your box dial outward to a server you own:

- [Pangolin *self-hosted tunneled reverse proxy on your own VPS*](/pangolin)
- [NetBird *mesh VPN with a Reverse Proxy feature for public services*](/netbird)
{.links-list}

> Using the **NetBird Reverse Proxy**? Self-hosted deployments must use **Traefik** as the external proxy, since it is the only one supporting the TLS passthrough the feature needs. NPM will not work in front of it, and the feature is still in beta. Test a large image push through it early.
{.is-warning}

# 4 · Container Registry

Forgejo ships an OCI-compliant registry at the same hostname as the web UI. Nothing extra to install.

## 4.1 Push an image

```bash
docker login git.serversatho.me
docker tag myapp:latest git.serversatho.me/yourname/myapp:1.0.0
docker push git.serversatho.me/yourname/myapp:1.0.0
```

Authenticate with a **Personal Access Token** (Settings → Applications) with the `write:package` scope, not your password. Required if you have 2FA on.

## 4.2 The hostname is permanent

Whatever hostname you chose is now baked into the image name itself. `git.serversatho.me/yourname/myapp` is not the same artifact as `registry.example.com/yourname/myapp`, and once other people put that string in their compose files, you cannot take it back.




# 5 · Sign in with GitHub

## 5.1 Create the OAuth app on GitHub

1. GitHub → **Settings** → **Developer settings** → **OAuth Apps** → **New OAuth App**
2. **Homepage URL**: `https://git.serversatho.me`
3. **Authorization callback URL**: `https://git.serversatho.me/user/oauth2/github/callback`
4. Save the **Client ID** and generate a **Client Secret**

## 5.2 Add it to Forgejo

**Site Administration** → **Identity & Access** → **Authentication Sources** → **Add Authentication Source**

| Setting | Value |
|---------|-------|
| Authentication Type | OAuth2 |
| Authentication Name | `github` |
| OAuth2 Provider | GitHub |
| Client ID | from step 5.1 |
| Client Secret | from step 5.1 |

The name must be exactly `github` to match the callback URL above.

`ALLOW_ONLY_EXTERNAL_REGISTRATION=true` in the compose file means new users can only sign up through GitHub, while existing local accounts including your admin still log in normally.

# 6 · Mirroring to GitHub


## 6.1 Set up a push mirror

In the repository on Forgejo: **Settings** → **Repository** → **Mirror Settings** → **Push Mirror**

| Setting | Value |
|---------|-------|
| Remote Repository URL | `https://github.com/youruser/yourrepo.git` |
| Authorization | your GitHub username |
| Password | GitHub PAT with `repo` scope |
| Sync when commits are pushed | Checked |

## 6.2 Turn off features on the GitHub side

On the GitHub repo: **Settings** → **Features**, and uncheck **Issues**, **Projects**, **Wiki** and **Discussions**. Under **Pull Requests**, make it clear in your README that PRs are not accepted there.

> **Push mirrors force-push.** If someone opens a pull request on your GitHub mirror and you merge it there, your next sync overwrites it and the work is gone. This is not merely awkward, it is a way to lose contributions. Turn the features off rather than leaving them on and hoping.
{.is-danger}

Then put a banner at the top of your README:

```markdown
> [!IMPORTANT]
> **Canonical home: [git.serversatho.me/yourname/yourrepo](https://git.serversatho.me/yourname/yourrepo)**
>
> Issues, pull requests, discussions and releases live there. The GitHub copy is a
> read-only mirror, synced automatically — anything opened against it will not be
> seen or answered.
>
> Sign in with your GitHub account, no new password needed.
```


## 6.3 A third copy, off your property

Add a second push mirror to [Codeberg](https://codeberg.org) or any other forge. It costs nothing and gives you a continuously updated offsite copy of every repository, without a backup job to babysit. It does not carry your issues or users, which is what section 8 is for.

> **Never delete the old GitHub repo.** Strip it down, archive it, but leave it up forever. The problem after a migration is never the code, it is that every inbound link from every blog post and forum answer now 404s. A repository whose only content is a signpost is worth more than a tidy namespace.
{.is-success}

# 7 · Forgejo Actions Runner
 
Forgejo Actions is workflow-compatible with GitHub Actions, so most existing YAML runs with minimal changes.
 
This section builds the runner in a dedicated, network-isolated Proxmox VM. 
 
## 7.1 Enable Actions
 
Add to your Forgejo environment and restart:
 
```yaml
      - FORGEJO__actions__ENABLED=true
      - FORGEJO__actions__DEFAULT_ACTIONS_URL=https://data.forgejo.org
```
 
Get a registration token from **Site Administration** → **Actions** → **Runners** → **Create new Runner**.
 
 
## 7.2 Build the CI VM
 
In Proxmox, **Create VM**, not an LXC. Docker-in-Docker inside LXC pushes you back to privileged containers, which defeats the purpose.
 
| Setting | Value |
|---------|-------|
| OS | Debian 13 netinst, minimal, no desktop |
| Cores | 4 |
| Memory | 8192 MB |
| Disk | 64 GB, thin provisioned |
| Network | VLAN tag for your CI network |
| QEMU Guest Agent | Enabled |
 
Install Docker and nothing else:
 
```bash
apt update && apt install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
  > /etc/apt/sources.list.d/docker.list
apt update && apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
 
 
## 7.4 Network isolation
 
Put the VM on its own VLAN. Using UniFi as the example:
 
| Direction | Action | Why |
|-----------|--------|-----|
| CI → Internet | Allow | Builds pull dependencies and base images |
| CI → NPM IP, port 443 only | Allow | The runner needs to reach Forgejo for jobs |
| CI → all other internal networks | **Drop** | Nothing else is its business |
| Any → CI | Drop | Nothing needs to initiate into it |
 
Rule order matters: the specific allow must sit **above** the drop-all. The runner dials outward, so it never needs an inbound rule.
 
Verify rather than assume. From inside the VM, `curl -k https://<truenas-ip>` should time out and `curl https://git.serversatho.me` should succeed. 

## 7.5 Deploy the runner
 
Inside the VM:
 
```yaml
services:
  forgejo-runner:
    image: code.forgejo.org/forgejo/runner:9
    container_name: forgejo-runner
    depends_on:
      - forgejo-dind
    environment:
      - FORGEJO_INSTANCE_URL=https://git.serversatho.me
      - FORGEJO_RUNNER_REGISTRATION_TOKEN=CHANGE_ME_TOKEN
      - DOCKER_HOST=tcp://forgejo-dind:2375
    restart: unless-stopped
    volumes:
      - /opt/forgejo-runner/data:/data
 
  forgejo-dind:
    image: docker:dind
    container_name: forgejo-dind
    privileged: true
    command: ["dockerd", "-H", "tcp://0.0.0.0:2375", "--tls=false"]
    environment:
      - DOCKER_TLS_CERTDIR=
    restart: unless-stopped
    volumes:
      - /opt/forgejo-runner/dind:/var/lib/docker
```
 
1. Create the directories: `mkdir -p /opt/forgejo-runner/{data,dind}`
2. Paste the registration token from 7.1
3. Give the runner the label `untrusted` when it registers
4. Confirm it shows as **Idle** under **Site Administration** → **Actions** → **Runners**

## 7.6 The two-runner split
 
The VM protects your data. This protects your users, and it is the half people skip.
 
Register a **second** runner, on TrueNAS via Dockge, using the same compose as 7.5 with `/mnt/tank/configs/forgejo-runner/` paths. Give it the label `publish`. It holds your registry token, and it only ever runs on tag pushes, which no outside contributor can perform.
 
| Runner | Where | Secrets | Triggers on |
|--------|-------|---------|-------------|
| `untrusted` | The CI VM | **None** | Pull requests, including forks |
| `publish` | TrueNAS via Dockge | Registry token | Tag pushes only |
 
```yaml
# Runs on every PR. Builds and tests. Cannot publish anything.
name: test
on: [pull_request]
jobs:
  test:
    runs-on: untrusted
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t testbuild .
```
 
```yaml
# Runs only when you push a tag, which only you can do.
name: release
on:
  push:
    tags: ['v*']
jobs:
  publish:
    runs-on: publish
    steps:
      - uses: actions/checkout@v4
      - run: echo "${{ secrets.REGISTRY_TOKEN }}" | docker login git.serversatho.me -u yourname --password-stdin
      - run: docker build -t git.serversatho.me/yourname/myapp:${{ github.ref_name }} .
      - run: docker push git.serversatho.me/yourname/myapp:${{ github.ref_name }}
```
 
The `publish` runner sits on the NAS with a live token because it never executes code you did not write. Untrusted input goes to the VM. Your own tagged releases go to the NAS. The boundary is what the runner *runs*, not where it sits.
 
A contributor can open any pull request they like. It builds credential-free in an isolated VM. Publishing needs a tag push they cannot make.
 
 
## 7.7 Reset the VM regularly
 
Each job runs in a fresh container, but the DinD daemon persists between jobs, so one compromised build could poison the next one's cache.
 
1. Get the VM to a clean working state
2. Take a Proxmox snapshot named `clean`
3. Schedule a nightly rollback on the Proxmox host:
```bash
# crontab -e on the Proxmox node, VMID 120 as an example
0 4 * * * /usr/sbin/qm rollback 120 clean && /usr/sbin/qm start 120
```
 
Also turn on the Forgejo setting requiring manual approval before workflows run on pull requests from first-time contributors.
 
## 7.8 If you never accept outside contributions
 
If a repository is only ever built from code you wrote, the CI VM is more than you need and the `publish` runner on TrueNAS can do everything. The realistic threat there is a compromised upstream dependency in your own build, which DinD contains reasonably well.

# <img src="/youtube.png" class="tab-icon"> 8 · Video 
https://youtu.be/O_kpayAlRZA
 
