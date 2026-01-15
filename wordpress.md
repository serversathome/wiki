---
title: Wordpress
description: A guide to installing Wordpress on TrueNAS and in docker via compose
published: true
date: 2026-01-15T15:32:38.071Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:42.750Z
---

![](/wordpress.png)

# What is Wordpress?
WordPress is a web content management system. It was originally created as a tool to publish blogs but has evolved to support publishing other web content, including more traditional websites, mailing lists and Internet forum, media galleries, membership sites, learning management systems and online stores.


# Installation
# {.tabset}
## TrueNAS
1. Set your **Database Password** to something secure
1. Set your **Root Database Password** to something secure
1. In **Network Configuration → WebUI Port** set the **Port Bind Mode** to `Publish port on the host for external access`
1. Set the **Storage Configuration** to `Host Path` for **Wordpress Data Storage** and **Wordpress Maria DB Storage**


## Docker Compose

```yaml
services:

 wordpress:
   image: wordpress
   container_name: wordpress
   restart: unless-stopped
   ports:
     - 8081:80
   environment:
     WORDPRESS_DB_HOST: db
     WORDPRESS_DB_USER: exampleuser
     WORDPRESS_DB_PASSWORD: examplepass
     WORDPRESS_DB_NAME: exampledb
   volumes:
     - ./wordpress:/var/www/html

 db:
   image: mysql:8.0
   restart: unless-stopped
   container_name: wordpress-db
   environment:
     MYSQL_DATABASE: exampledb
     MYSQL_USER: exampleuser
     MYSQL_PASSWORD: examplepass
     MYSQL_RANDOM_ROOT_PASSWORD: '1'
   volumes:
     - ./db:/var/lib/mysql
```

1. Change all the default user names and passwords. Make sure when you change them in the wordpress service you mirror those changes in the db service. 
> 
> You can deploy as many of these containers as you want for each website you want to host. Be sure to change the external port and the container names for each one. 
{.is-info}


# Wordpress Configuration

The install process is straight forward. In order to secure your Wordpress site, remove all the default themes and plugins, adding back in only what you need. Also make sure to enable automatic updates for any themes or plug-ins you install. 

## Behind a Reverse Proxy

To run Wordpress behind a reverse proxy, go into the settings and change the URLs to your FQDN.

![](/screenshot_from_2024-12-06_06-58-08.png)

# Monitoring With Umami

If you are using [Umami](/umami) to monitor web traffic, install the Umami plugin. 

1. Navigate to Plugins > Add New Plugin > Search Plugins
![](/screenshot_from_2024-06-30_17-36-45.png)


1. Navigate to plugins > Integrate Umami > Settings
1. Use the information about the website you add to Umami to fill in this information:
![](/screenshot_from_2024-06-30_17-39-35.png)

