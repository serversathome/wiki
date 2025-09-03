---
title: Docusaurus
description: A guide to deploying Docusaurus in docker
published: true
date: 2025-09-03T11:02:55.656Z
tags: 
editor: markdown
dateCreated: 2025-08-27T08:38:39.465Z
---

# <img src="/docusaurus.png" class="tab-icon"> What is Docusaurus?
Docusaurus is an open-source static site generator designed for building documentation websites quickly and easily, primarily using Markdown. It allows users to create customizable sites while focusing on content creation, making it popular among software documentation teams.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Docusaurus


```yaml
services:

  dev:
    container_name: docusaurus-dev
    image: trevorbrunette/docusaurus:dev
    ports:
      - 3000:3000
    volumes:
      - /mnt/tank/configs/docusaurus:/opt/docusaurus
    healthcheck:
      test:
        - CMD
        - curl
        - -f
        - http://localhost:3000/
      interval: 30s
      timeout: 10s
      retries: 3
      # Give More Time To Start Up If Failed 
      start_period: 120s

  build:
    depends_on:
      dev:
        condition: service_healthy
    container_name: docusaurus-build
    image: trevorbrunette/docusaurus:build
    restart: no
    volumes:
      - /mnt/tank/configs/docusaurus:/opt/docusaurus
    healthcheck:
      test:
        - CMD
        - bash
        - -c
        - "[ -f /opt/docusaurus/index.html ]"
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 3s

  server:
    depends_on:
      - build
    container_name: docusaurus-serve
    image: trevorbrunette/docusaurus:serve
    ports:
      - 8082:8080
    volumes:
      - /mnt/tank/configs/docusaurus:/opt/docusaurus
    healthcheck:
      test:
        - CMD
        - curl
        - -f
        - http://localhost:8080/
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s


  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=0
      - PGID=0
      - TZ=America/New_York
      - PASSWORD=password
      - DEFAULT_WORKSPACE=/mnt/
    volumes:
      - /mnt/tank/configs/codeserver:/config
      - /mnt/tank/configs/docusaurus:/mnt
    ports:
      - 8443:8443
    restart: unless-stopped
```
> If your Dockge stack says **exited**, that is normal. The `build` container does not run constantly and will throw a false positive to Dockge even though all the other containers should read `healthy` or `running`.
{.is-info}


# 2 · How it Works

Docusaurs has both a `dev` site running on port `3000` and a `production` site which will run on port `8082`. It is generally unsafe to expose the `dev` server to the internet, so Docusaurus "builds" a production site from the dev site for the public.

Everytime you make a change to the files in `/mnt/tank/configs/docusaurus` the `dev` site will be updated immediately. However, the `prod` site will need to be rebuilt every time. 

## 2.1 Building a Production Site

To build the `prod` site after edits, press the **Start** button in Dockge, or restart the containers if you deployed them via CLI or some other container manager.


Once the `prod` site is built, point your reverse proxy at `http://{IP}:8082` to serve your Docusaurus site. 


## 2.2 Editing the Files

This compose stack comes with a **VS Code** container which will allow you to edit the files in the `/mnt/tank/configs/docusaurus` directory through a visual editor. Once the files are edited the changes will be reflected immediately on the `dev` server. 

To login to the code container, visit `http://{IP}:8443` and use the password `password`.

# 3 · Publishing to GitHub

GitHub can host your Docusaurus site on GitHub Pages. Follow the instructions below to publish your production site to GitHub.

## 3.2 Docusaurus Configuration
1. Open the `docusaurus.config.js` file
1. Set the `url` to `https://{github username}.github.io`
1. Chnage the `organizationName`to your GitHub org/user name
1. Change the `projectName` to your repo name

## 3.1 Permissions

We need to set up a Personal Access Token so we have permission to publish to our GitHub site.

1. Go to your GitHub account **Settings**
1. In the left sidebar, click on **Developer settings** all the way at the bottom
1. Click on **Personal access tokens → Fine-grained tokens**
1. Click on **Generate new token**
1. Give your token a name (e.g., docusaurus-deploy)
1. Set an expiration date for the token
1. Under **Repository access** choose your docusaurus repo
1. In the **Permissions** section, click **Add Permissions** 
	a. Check the **Contents** box and give **Read and write** permissions
1. Click **Generate token** at the bottom
1. Copy the generated token immediately. You will not be able to see it again.

## 3.3 Initialize the Repo
1. Navigate to the directory where the config files are (`/mnt/tank/configs/docusaurus`)
1. Run the following commands one at a time:
    ```bash
    git init
    ```
    ```bash
    git remote add origin https://github.com/{USERNAME}/docs.git
    ```
    ```bash
    git remote set-url origin https://<YOUR_PERSONAL_ACCESS_TOKEN>@github.com/{USERNAME}/{REPONAME}.git
    ```
    ```bash
    git add .
    ```
    ```bash
    git commit -m "Initial commit for Docusaurus site"
    ```
    ```bash
    git push --set-upstream origin main
    ```

## 3.4 Publishing
1. Either hit the **Start** button in Dockge or restart the stack to tirgger a build
1. Run the following command from the TrueNAS shell to publish the site to GitHub:
```bash
docker start docusaurus-build && docker exec docusaurus-build npm run deploy
```

## 3.1 GitHub Pages

Set up a repo and add Pages following the steps below:

1. Once inside your repo, click **Settings → Pages**
1. Under **Branch**, set the first dropdown box to **Main** and click the **Save** button directly next to it



