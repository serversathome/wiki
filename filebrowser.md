---
title: File Browser
description: A guide to deploy the File Browser Quantum replacement in docker
published: true
date: 2025-07-16T16:44:24.803Z
tags: 
editor: markdown
dateCreated: 2025-06-11T23:53:09.996Z
---

# ![](/filebrowser-quantum.png){class="tab-icon"} What is File Browser Quantum?
File Browser provides a file managing interface within a specified directory and it can be used to upload, delete, preview, rename and edit your files. 

# This is not the old File Browser
The popular file browser repository is no longer accepting pull requests and is [maintenance only mode](https://github.com/filebrowser/filebrowser/discussions/4906#discussioncomment-13436994).

Some kind human has forked the repo and built a much better version (that is still in beta). His repo is [here](https://github.com/gtsteffaniak/filebrowser).

I have tested this and there is clearly still stuff missing but in terms of file management it works great!

# <img src="/docker.png" class="tab-icon"> 1 · Deploy File Browser
```yaml
services:
  filebrowser:
    environment:
      FILEBROWSER_CONFIG: data/config.yaml
    volumes:
      - /mnt:/srv
      - /mnt/tank/configs/filebrowser:/home/filebrowser/data
    ports:
      - 7999:80
    image: gtstef/filebrowser:latest
    restart: unless-stopped
    container_name: filebrowser
```

Upon first run of this it will fail to launch. You will need to shell into the `/mnt/tank/configs/filebrowser` directory and add a file called `config.yaml` with these contents:

```yaml
server:
  port: 80
  baseURL:  "/"
  logging:
    - levels: "info|warning|error"
  sources:
    - path: "/srv"
      config:
        defaultEnabled: true # add source for all users by default 
userDefaults:
  preview:
    image: true
    popup: true
    video: false
    office: false
    highQuality: false
  darkMode: true
  disableSettings: false
  singleClick: true
  permissions:
    admin: true
    modify: true
    share: true
    api: true
```

There are many more options for this config file. [See them all here](https://github.com/gtsteffaniak/filebrowser/wiki/Full-Config-Example).

# 2 · Logging In
The default user is `admin` and the default password is `admin`.