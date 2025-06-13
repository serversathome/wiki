---
title: Recyclarr
description: A guide to installing Recyclarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-06-13T03:08:33.315Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:34:38.606Z
---

![](https://wiki.hydrology.cc/recyclarr.png)

# What is Recyclarr?

Recyclarr is a command-line application that will automatically synchronize recommended settings from the TRaSH guides to your Sonarr/Radarr instances.

# Installation
# {.tabset}
## Docker Compose

```yaml
services:
  recyclarr:
    image: ghcr.io/recyclarr/recyclarr:latest
    user: 568:568
    container_name: recyclarr
    restart: unless-stopped
    build:
      context: .
      args:
        - TARGETARCH=amd64
    volumes:
      - /mnt/tank/configs/recyclarr:/config
    environment:
      CRON_SCHEDULE: 0 0 * * *
      TZ: America/New_York
```

### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/recyclarr
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

## TrueNAS

![](https://wiki.hydrology.cc/screenshot_from_2023-12-12_09-34-37.png)

- Change the **Config Storage Type** to **Host Path** as per the [Folder-Structure](/Folder-Structure) guide.

# Recyclarr Configuration

Recyclarr is primarily a command-line tool. The good news is once its set up, it doesn't require further intervention (it is configured to update itself daily).

## Recyclarr.yml

This is the file within Recyclarr which tells it what to sync to Sonarr/Radarr. I have pre-populated it to sync HD and UHD to Radarr and Sonarr (v4) as well as remove all the old custom formats that come preloaded with Sonarr/Radarr.

> The assumption is your devices are not compatible with HDR or Dolby Vision. If they are, uncomment out the lines in the .yml file below where instructed
{.is-info}

Copy the code below into a text editor and edit the lines for your specific instance URL and API key. Note, leave a space between the colon and the API key or the sync will fail.

<details><summary><strong>Show me the code!</strong></summary>

```yaml
sonarr:
  web-1080p-v4:
    base_url: http://sonarr:8989
    api_key: 
    delete_old_custom_formats: true
    replace_existing_custom_formats: true
    include:
      # Comment out any of the following includes to disable them
      - template: sonarr-quality-definition-series
      - template: sonarr-v4-quality-profile-web-1080p
      - template: sonarr-v4-custom-formats-web-1080p
      - template: sonarr-v4-quality-profile-web-2160p
      - template: sonarr-v4-custom-formats-web-2160p

# Custom Formats: https://recyclarr.dev/wiki/yaml/config-reference/custom-formats/
    custom_formats:
      # HDR Formats
      - trash_ids:
          # Comment out the next line if you and all of your users' setups are fully DV compatible
          - 9b27ab6498ec0f31a3353992e19434ca # DV (WEBDL)
          # HDR10Plus Boost - Uncomment the next line if any of your devices DO support HDR10+
          # - 0dad0a507451acddd754fe6dc3a7f5e7 # HDR10Plus Boost
        assign_scores_to:
          - name: WEB-2160p


      # Optional
      - trash_ids:
           - 32b367365729d530ca1c124a0b180c64 # Bad Dual Groups
           - 82d40da2bc6923f41e14394075dd4b03 # No-RlsGroup
           - e1a997ddb54e3ecbfe06341ad323c458 # Obfuscated
           - 06d66ab109d4d2eddb2794d21526d140 # Retags
           - 1b3994c551cbb92a2c781af061f4ab44 # Scene
        assign_scores_to:
          - name: WEB-2160p

      - trash_ids:
          # Uncomment the next six lines to allow x265 HD releases with HDR/DV
          # - 47435ece6b99a0b477caf360e79ba0bb # x265 (HD)
        # assign_scores_to:
          # - name: WEB-2160p
            # score: 0
      # - trash_ids:
          # - 9b64dff695c2115facf1b6ea59c9bd07 # x265 (no HDR/DV)
        assign_scores_to:
          - name: WEB-2160p

      - trash_ids:
          - 2016d1676f5ee13a5b7257ff86ac9a93 # SDR
        assign_scores_to:
          - name: WEB-2160p
            # score: 0 # Uncomment this line to enable SDR releases

      # Optional
      - trash_ids:
           - 32b367365729d530ca1c124a0b180c64 # Bad Dual Groups
           - 82d40da2bc6923f41e14394075dd4b03 # No-RlsGroup
           - e1a997ddb54e3ecbfe06341ad323c458 # Obfuscated
           - 06d66ab109d4d2eddb2794d21526d140 # Retags
           - 1b3994c551cbb92a2c781af061f4ab44 # Scene
        assign_scores_to:
          - name: WEB-1080p

      - trash_ids:
          # Uncomment the next six lines to allow x265 HD releases with HDR/DV
          # - 47435ece6b99a0b477caf360e79ba0bb # x265 (HD)
        # assign_scores_to:
          # - name: WEB-1080p
            # score: 0
      # - trash_ids:
          # - 9b64dff695c2115facf1b6ea59c9bd07 # x265 (no HDR/DV)
        assign_scores_to:
          - name: WEB-1080p

# Configuration specific to Radarr.
radarr:
 uhd-bluray-web:
    base_url: http://radarr:7878
    api_key: 
    delete_old_custom_formats: true
    replace_existing_custom_formats: true
    include:
     # Comment out any of the following includes to disable them
     - template: radarr-quality-definition-movie
     - template: radarr-quality-profile-uhd-bluray-web
     - template: radarr-custom-formats-uhd-bluray-web
     - template: radarr-quality-definition-movie
     - template: radarr-quality-profile-hd-bluray-web
     - template: radarr-custom-formats-hd-bluray-web

# Custom Formats: https://recyclarr.dev/wiki/yaml/config-reference/custom-formats/
    custom_formats:
     # Audio
     - trash_ids:
         # Uncomment the next section to enable Advanced Audio Formats
         # - 496f355514737f7d83bf7aa4d24f8169 # TrueHD Atmos
         # - 2f22d89048b01681dde8afe203bf2e95 # DTS X
         # - 417804f7f2c4308c1f4c5d380d4c4475 # ATMOS (undefined)
         # - 1af239278386be2919e1bcee0bde047e # DD+ ATMOS
         # - 3cafb66171b47f226146a0770576870f # TrueHD
         # - dcf3ec6938fa32445f590a4da84256cd # DTS-HD MA
         # - a570d4a0e56a2874b64e5bfa55202a1b # FLAC
         # - e7c2fcae07cbada050a0af3357491d7b # PCM
         # - 8e109e50e0a0b83a5098b056e13bf6db # DTS-HD HRA
         # - 185f1dd7264c4562b9022d963ac37424 # DD+
         # - f9f847ac70a0af62ea4a08280b859636 # DTS-ES
         # - 1c1a4c5e823891c75bc50380a6866f73 # DTS
         # - 240770601cc226190c367ef59aba7463 # AAC
         # - c2998bd0d90ed5621d8df281e839436e # DD
       assign_scores_to:
         - name: UHD Bluray + WEB

     # Movie Versions
     - trash_ids:
         - 9f6cbff8cfe4ebbc1bde14c7b7bec0de # IMAX Enhanced
       assign_scores_to:
         - name: UHD Bluray + WEB
           # score: 0 # Uncomment this line to disable prioritised IMAX Enhanced releases

     # Optional
     - trash_ids:
         # Comment out the next line if you and all of your users' setups are fully DV compatible
         - 923b6abef9b17f937fab56cfcf89e1f1 # DV (WEBDL)
         # HDR10Plus Boost - Uncomment the next line if any of your devices DO support HDR10+
         # - b17886cb4158d9fea189859409975758 # HDR10Plus Boost
       assign_scores_to:
         - name: UHD Bluray + WEB

     - trash_ids:
         - 9c38ebb7384dada637be8899efa68e6f # SDR
       assign_scores_to:
         - name: UHD Bluray + WEB
           # score: 0 # Uncomment this line to allow SDR releases

     - trash_ids:
         - 9f6cbff8cfe4ebbc1bde14c7b7bec0de # IMAX Enhanced
       assign_scores_to:
         - name: HD Bluray + WEB
           # score: 0 # Uncomment this line to disable prioritised IMAX Enhanced releases
```
</details>
To download this script use the command:

```bash
wget https://raw.githubusercontent.com/imjustleaving/ServersatHome/refs/heads/main/recyclarr.yml
```

## Syncing Recyclarr

1. Start by entering the shell by clicking the button below:

![](https://wiki.hydrology.cc/recyclarrshell.png)

2. Enter the command 
```bash
recyclarr config create
```

3. Now we have the enter a series of commands into the shell
> Enter these commands one at a time using **Option 1** from where you are currently or **Option 2** from the TrueNAS Shell
{.is-info}

# {.tabset}
## Option 1
 1. Enter this command:
```bash
vi /config/recyclarr.yml
```
1.  Press the <kbd>INS</kbd> key (insert)
1.  Copy the text from the recyclarr.yml section above. Once it is in your clipboard, hit <kbd>Shift+INS</kbd> to paste
1.  Add your API Keys for Sonarr/Radarr
1.  Hit <kbd>Esc</kbd>
1.  Type `:wq!` then hit <kbd>Enter</kbd>
1.  Type `recyclarr sync`

## Option 2

1.  Using the TrueNAS Shell, navigate to the hostpath directory where Recyclarr's data is stored (example: `/mnt/tank/configs/recyclarr/`)
2.  type `nano recyclarr.yml`
3.  Copy the Recyclarr.yml config file from above and paste it into the text editor using <kbd>Shift+INS</kbd>
4.  Add your API Keys for Sonarr/Radarr
5.  Hit <kbd>Ctrl+X</kbd> then <kbd>Y</kbd> then <kbd>Enter</kbd> to save and quit.
6.  Shell back into the Recyclarr container and type `recyclarr sync`

These steps effectively create a .yml config file, enter into it, paste our text into it, save and exit, then sync our changes into Radarr/Sonarr.

# Video Walkthrough
https://youtu.be/sIvBG9SbIQo

![](/2025-01-30-mastering-radarr--recyclarr-a--promo-card.png)

[Watch it on Patreon!](https://www.patreon.com/posts/mastering-radarr-121113567?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)


![](/2025-01-30-mastering-sonarr--recyclarr-a--promo-card.png)
[Watch it on Patreon!](https://www.patreon.com/posts/mastering-sonarr-121115716?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)