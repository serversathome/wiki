---
title: Wiki.js
description: A guide to installing Wiki.js in docker via compose
published: true
date: 2025-06-08T18:39:43.326Z
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
