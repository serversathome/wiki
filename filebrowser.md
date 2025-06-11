---
title: File Browser
description: A guide to deploy the File Browser Quantum replacement in docker
published: true
date: 2025-06-11T23:57:25.563Z
tags: 
editor: markdown
dateCreated: 2025-06-11T23:53:09.996Z
---

![408705763-59986a2a-f960-4536-aa35-4a9a7c98ad48.png](/408705763-59986a2a-f960-4536-aa35-4a9a7c98ad48.png)

# What is File Browser?
File Browser provides a file managing interface within a specified directory and it can be used to upload, delete, preview, rename and edit your files. 

# This is not the old File Browser
The popular file browser repository is no longer accepting pull requests and is [maintenance only mode](https://github.com/filebrowser/filebrowser/discussions/4906#discussioncomment-13436994).

Some kind human has forked the repo and built a much better version (that is still in beta). His repo is [here](https://github.com/gtsteffaniak/filebrowser).

I have tested this and there is clearly still stuff missing but in terms of file management it works great!

# Docker Compose
```yaml
services:
  filebrowser:
    stdin_open: true
    tty: true
    volumes:
      - /mnt/:/srv
    ports:
      - 7999:80
    image: gtstef/filebrowser:beta
    restart: unless-stopped
    container_name: filebrowser
```

# Logging In
The default user is `admin` and the default password is `admin`.