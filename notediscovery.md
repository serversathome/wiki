---
title: Note Discovery
description: A guide to deploying Note Discovery
published: true
date: 2025-12-02T12:13:39.538Z
tags: 
editor: markdown
dateCreated: 2025-12-02T12:13:39.538Z
---

# <img src="/notediscovery.png" class="tab-icon"> What is Note Discovery?
NoteDiscovery is a lightweight, self-hosted note-taking application that puts you in complete control of your knowledge base. Write, organize, and discover your notes with a beautiful, modern interface—all running on your own server.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Note Discovery
1. Navigate to your `stacks` directory
1. Run these commands in the TrueNAS shell as root:
    ```bash
    git clone https://github.com/gamosoft/NoteDiscovery
    mv NoteDiscovery notediscovery
    mkdir notediscovery/data/_templates
    rsync -avhz notediscovery/documentation/templates/ notediscovery/data/_templates/
    ```
1. Refresh you Dockge page
1. Start the stack