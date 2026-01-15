---
title: PhotoPrism
description: A guide to deploying PhotoPrism
published: true
date: 2026-01-15T15:30:43.462Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:08.400Z
---

# ![](/photoprism.png){class="tab-icon"} What is PhotoPrism?

PhotoPrism® is an AI-Powered Photos App for the Decentralized Web. It makes use of the latest technologies to tag and find pictures automatically without getting in your way. 

# 1 · Deploy PhotoPrism
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Site URL** to either your local IP or a FQDN
1. Set the **Admin Password** 
1. Set the **Photoprism Import Storage** to **Host Path**
1. Set the **Photoprism Storage** to **Host Path**
1. Set the **Photoprism Originals Storage** to **Host Path**


## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  photoprism:
    image: photoprism/photoprism:latest
    restart: unless-stopped
    stop_grace_period: 15s
    depends_on:
      - mariadb
    security_opt:
      - seccomp:unconfined
      - apparmor:unconfined
    ports:
      - "2342:2342"
    ## https://docs.photoprism.app/getting-started/config-options/
    environment:
      PHOTOPRISM_ADMIN_USER: "admin"                 
      PHOTOPRISM_ADMIN_PASSWORD: "insecure"          
      PHOTOPRISM_AUTH_MODE: "password"              
      PHOTOPRISM_DISABLE_TLS: "false"                
      PHOTOPRISM_DEFAULT_TLS: "true"               
      PHOTOPRISM_DEFAULT_LOCALE: "en"               
      PHOTOPRISM_PLACES_LOCALE: "local"           
      PHOTOPRISM_SITE_URL: ${HOST}
      PHOTOPRISM_SITE_TITLE: "PhotoPrism"
      PHOTOPRISM_SITE_CAPTION: "AI-Powered Photos App"
      PHOTOPRISM_SITE_DESCRIPTION: ""               
      PHOTOPRISM_SITE_AUTHOR: ""          
      PHOTOPRISM_LOG_LEVEL: "info"                  
      PHOTOPRISM_READONLY: "false"                 
      PHOTOPRISM_EXPERIMENTAL: "false"            
      PHOTOPRISM_DISABLE_CHOWN: "false"              
      PHOTOPRISM_DISABLE_WEBDAV: "false"             
      PHOTOPRISM_DISABLE_SETTINGS: "false"        
      PHOTOPRISM_DISABLE_TENSORFLOW: "false"      
      PHOTOPRISM_DISABLE_FACES: "false"              
      PHOTOPRISM_DISABLE_CLASSIFICATION: "false"    
      PHOTOPRISM_DISABLE_VECTORS: "false"        
      PHOTOPRISM_DISABLE_RAW: "false"              
      PHOTOPRISM_RAW_PRESETS: "false"             
      PHOTOPRISM_SIDECAR_YAML: "true"                
      PHOTOPRISM_BACKUP_ALBUMS: "true"              
      PHOTOPRISM_BACKUP_DATABASE: "true"           
      PHOTOPRISM_BACKUP_SCHEDULE: "daily"         
      PHOTOPRISM_INDEX_SCHEDULE: ""                 
      PHOTOPRISM_AUTO_INDEX: 300                  
      PHOTOPRISM_AUTO_IMPORT: -1                    
      PHOTOPRISM_DETECT_NSFW: "false"              
      PHOTOPRISM_UPLOAD_NSFW: "true"                
      PHOTOPRISM_UPLOAD_ALLOW: ""                   
      PHOTOPRISM_UPLOAD_ARCHIVES: "true"            
      PHOTOPRISM_UPLOAD_LIMIT: 5000               
      PHOTOPRISM_ORIGINALS_LIMIT: 5000              
      PHOTOPRISM_HTTP_COMPRESSION: "gzip"               
      PHOTOPRISM_DATABASE_DRIVER: "mysql"           
      PHOTOPRISM_DATABASE_SERVER: "mariadb:3306"   
      PHOTOPRISM_DATABASE_NAME: "photoprism"        
      PHOTOPRISM_DATABASE_USER: "photoprism"        
      PHOTOPRISM_DATABASE_PASSWORD: "insecure"      
      PHOTOPRISM_INIT: "https tensorflow"           
      PHOTOPRISM_VISION_API: "false"              
      PHOTOPRISM_VISION_URI: ""                      
      PHOTOPRISM_VISION_KEY: ""                     
    working_dir: "/photoprism"
    volumes:
      - ${CONFIG_PATH}/originals:/photoprism/originals              
      - ${CONFIG_PATH}/import:/photoprism/import           
      - ${CONFIG_PATH}/storage:/photoprism/storage                


  mariadb:
    image: mariadb:11
    restart: unless-stopped
    stop_grace_period: 15s
    security_opt: 
      - seccomp:unconfined
      - apparmor:unconfined
    command: --innodb-buffer-pool-size=512M --transaction-isolation=READ-COMMITTED --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --max-connections=512 --innodb-rollback-on-timeout=OFF --innodb-lock-wait-timeout=120
    volumes:
      - ${CONFIG_PATH}/database:/var/lib/mysql
    environment:
      MARIADB_AUTO_UPGRADE: "1"
      MARIADB_INITDB_SKIP_TZINFO: "1"
      MARIADB_DATABASE: "photoprism"
      MARIADB_USER: "photoprism"
      MARIADB_PASSWORD: "insecure"
      MARIADB_ROOT_PASSWORD: "insecure"
```

> For information about all of the environment variables, [see the docs](https://docs.photoprism.app/getting-started/config-options/)
{.is-info}


### env File
```yaml
HOST=http://10.99.0.242:2342
CONFIG_PATH=/mnt/tank/configs/photoprism
```