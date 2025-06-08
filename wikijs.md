---
title: Wiki.js
description: A guide to installing Wiki.js in docker via compose
published: true
date: 2025-06-08T20:31:31.439Z
tags: 
editor: markdown
dateCreated: 2024-08-22T11:00:30.633Z
---

![](/wikijs-full-inverted-3126313965.png)

# What is Wiki.js?

**Wiki.js** is an open source wiki software that works on any platform and database. It offers features such as authentication, media, themes, integrations and more.

# Docker Compose

```yaml
services:
 db:
   image: postgres:15-alpine
   container_name: wikijs-db
   environment:
     POSTGRES_DB: wiki
     POSTGRES_PASSWORD: wikijsrocks
     POSTGRES_USER: wikijs
   logging:
     driver: none
   restart: unless-stopped
   volumes:
     - /mnt/tank/configs/wikijs:/var/lib/postgresql/data
 wiki:
   image: ghcr.io/requarks/wiki:latest
   container_name: wikijs
   depends_on:
     - db
   environment:
     DB_TYPE: postgres
     DB_HOST: db
     DB_PORT: 5432
     DB_USER: wikijs
     DB_PASS: wikijsrocks
     DB_NAME: wiki
   restart: unless-stopped
   ports:
     - 3000:3000
```

1. Change all the default user names and passwords. Make sure when you change them in the wiki service you mirror those changes in the db service. 
> 
> You can deploy as many of these containers as you want for each wiki you want to host. Be sure to change the external port for each one.
{.is-info}

# Making it Beautiful

The strength of any editor is the Markdown function. The easiest way to start with Wiki.js is with the WYSIWYG editor, which operates like Microsoft Word. However, all of the most powerful functions are only availble using the markdown editor. 

## Converting Editors

To change a page from WYSIWYG to the Markdown editor:
1. Navigate to the Page Actions button while viewing the page you would like to convert
1. Click **Convert**
1. Select **Markdown** and hit **Convert**
> 
> A note when you do this, if you convert back to WYSIWYG all the formatting will be lost from Markdown
{.is-warning}

> All the markdown instructions can be found [here](https://docs.requarks.io/en/editors/markdown)
{.is-success}

## Navigation Icons

By default, Wiki.js uses the Material Designs icon set for naviagtion. The best way to do navigation in my opinion is to go to **Administration Settings** ⚙️ → **Navigation** ➤ and set the **Navigation Mode** to **Custom Naviagtion** and change the default *mdi-chevron-right* to something from [here](https://pictogrammers.com/library/mdi/).

# Making YouTube Links Work
To show the video player when inserting YouTube links, you must make the following changes:
1. Navigate to the **Administration Settings** ⚙️
1. Navigate to the **Theme** tab 🎨
1. In the **Code Injection** on the right, paste this in the **CSS Override** field:
```css
.responsive-embed {
  position: relative;
  padding-bottom: 56.2%;
  height: 0;
  margin: 10px 0;
  overflow: hidden
}
.responsive-embed iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border-radius: 15px
}
```
4. In the **HTML Injection** field paste the following:
```html
<script type="text/javascript" defer>
  const rxYoutube = /^.*^((?:https?:)?\/\/)?((?:www|m)\.)?((?:youtube\.com|youtu.be))(\/(?:[\w\-]+\?v=|embed\/|v\/|shorts\/)?)([\w\-]+)(\S+)?$/

 const rxScreencast = /^.*^((?:https?:)?\/\/)?(www\.)?(screencast\.com)(\/users)\/([a-z0-9_-]+)\/folders\/([a-z0-9%_-]+)\/media\/([a-z0-9_-]+)(?:\/)?$/im

  window.boot.register('vue', () => {
    window.onload = () => {
      document.querySelectorAll('.contents oembed, .contents a').forEach(elm => {
        const url = elm.hasAttribute('url') ? elm.getAttribute('url') : elm.getAttribute('href')
        let newElmHtml = null
       
        const ytMatch = url.match(rxYoutube)
        const scMatch = url.match(rxScreencast)

 
        if (ytMatch) {
          newElmHtml = `<iframe id="ytplayer" type="text/html" width="640" height="360" src="https://www.youtube-nocookie.com/embed/${ytMatch[5]}" frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>`
        
      } else if(scMatch) {
        newElmHtml = `<iframe id="scplayer" type="text/html" width="640" height="360" src="${url}/embed" frameborder="0" allowfullscreen></iframe>`

        } else if (url.endsWith('.mp4')) {
          newElmHtml = `<video controls autostart="0" name="media" width="640" height="360"><source src="${url}" type="video/mp4"></video>`
        } else {
         return
         }

        const newElm = document.createElement('div') 
        newElm.classList.add('responsive-embed')
        newElm.insertAdjacentHTML('beforeend', newElmHtml)
        elm.replaceWith(newElm)
      })
    }  
  })
</script>
```

# Syncing With GitHub

[GitHub](https://www.github.com) is the most popular git source control provider.

## Generate a new key

1. Open Terminal.
2. Enter the command:
   ```bash
   ssh-keygen -t rsa -b 4096
	 ```
3. When prompted to save the generated file, enter a path which can be accessed by Wiki.js *(e.g. `/etc/wiki/github.pem`)* and press **Enter**.
4. Leave the passphrase empty and press **Enter** twice. Password-protected keys will NOT work.

> On Windows, you can use [Git Bash](https://git-scm.com/download/win) or Windows Subsystem for Linux (WSL) distributions like [Ubuntu for Windows](https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6) to run the commands above. You can also generate keys manually using the [puttygen](https://www.ssh.com/ssh/putty/download) utility.
{.is-info}

## Add the key to GitHub

1. Create a new GitHub repository.
2. Click on the **Settings** tab.
3. Click on the **Deploy Keys** in the left navigation menu.
4. Click the **Add deploy key** button.
5. Enter a name of your choice for this key (e.g. wiki) and paste the contents of the public key generated earlier. *(file ending in `.pub`)*
6. Make sure the **Allow write access** is checked.
7. Click the **Add key** button.

## Configure Wiki.js

1. In the Administration Area, click on **Storage** in the left navigation menu.
2. Make sure the **Git** storage target is checked.
3. Click on the **Git** tab.
4. Enter the following settings:
   - Authentication Type: `ssh`
   - Repository URI: *On your GitHub repository page, in the **Code** tab, click on the **Clone or download** green button and copy the URI shown below **Clone with SSH**.*
   - Branch: `main`
   - SSH Private Key Path: *The path to your private key generated earlier.*
   - Username: *Empty*
   - Password: *Empty*
   - Default Author Email: *Should match your GitHub account email.*
   - Default Author Name: *Should match your GitHub account name.*
   - Local Repository Path: *Choose where the repo will be cloned locally or leave the default `./data/repo` value.*
   - Verify SSL Certificate: **On**
5. Set the **Sync Direction** to **Bi-directional**.
6. Set the **Sync Schedule** to **5 minutes**.
7. Click the **Apply Changes** button at the top of the page.
8. Wait for the **Status** panel to update. A new entry for **Git** should appear in green. If the bar is red, it means you have an error in your configuration. Go back to the Git tab, fix the error and try again.

## Adding the Edit Button
Add the following code to the following sections:

### CSS Override

```css
.edit-on-github-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: #24292e;
  color: white !important;
  padding: 8px 16px;
  border-radius: 4px;
  text-decoration: none;
  font-size: 14px;
  z-index: 9999;
  border: 1px solid #444;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
  transition: all 0.3s ease;
}
.edit-on-github-btn:hover {
  background: #0366d6;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.3);
}
```

### Head HTML Injection
```html
<script>
  function addEditButton() {
    // Clean the path - removes /en/ and fixes for GitHub
    const fullPath = window.location.pathname;
    const path = fullPath
      .replace(/^\/en\//, '')      // Remove /en/ prefix
      .replace(/^\//, '')          // Remove leading slash
      .replace(/-/g, '/')          // Convert hyphens to slashes
      .replace(/(\.md)?\/?$/, ''); // Remove .md and trailing slashes
    
    // Create button
    const btn = document.createElement('a');
    btn.className = 'edit-on-github-btn';
    btn.href = `https://github.com/GitHubName/RepoName/edit/main/${path}.md`;
    btn.target = '_blank';
    btn.textContent = '✏️ Edit on GitHub';
    
    // Add to body (fixed position doesn't need specific container)
    document.body.appendChild(btn);
  }

  // Run on initial load
  document.addEventListener('DOMContentLoaded', addEditButton);
  
  // Run on Turbo Drive/AJAX navigation if used
  if (typeof Turbo !== 'undefined') {
    document.addEventListener('turbo:load', addEditButton);
  }
  if (typeof pjax !== 'undefined') {
    document.addEventListener('pjax:complete', addEditButton);
  }
</script>
```
> In the `btn.href` line 14 you need to put in the correct GitHub Name and Repo for your wiki
{.is-warning}


### Body HTML Injection
```html
<noscript>
  <a href="https://github.com/serversathome/wiki/" 
     class="edit-on-github-btn"
     target="_blank">
    Edit on GitHub
  </a>
</noscript>
```
