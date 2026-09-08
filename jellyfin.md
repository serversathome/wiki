---
title: Jellyfin
description: A guide to deploying Jellyfin on TrueNAS as well as Docker
published: true
date: 2026-01-18T10:55:31.854Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:36.259Z
---

# ![](/jellyfin.png){class="tab-icon"} What is Jellyfin?

Jellyfin enables you to collect, manage, and stream your media. Run the Jellyfin server on your system and gain access to the leading free-software entertainment system, bells and whistles included.

<div class="glance">
  <div><span>Port</span><b><code>8096</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Hardware accel</span><b>Nvidia, Intel or AMD</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://jellyfin.org/docs/general/installation/advanced/truenas/"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://apps.truenas.com/resources/deploy-jellyfin"><i class="mdi mdi-server"></i>TrueNAS app</a>
    <a href="https://youtu.be/VNhsR2lpntc"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

# 1 · Deploy Jellyfin
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  jellyfin:
    container_name: jellyfin
    image: lscr.io/linuxserver/jellyfin:latest
    runtime: nvidia
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=all
    ports:
      - '8096:8096'
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/jellyfin:/config
      - /mnt/tank/media:/media
```
> Note this stack enables nVidia GPUs. If you don't need this, remove all the lines with `nvidia` in them.
{.is-info}

### Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.
- **Volumes**: The container structure follows a common-sense naming convention, storing configurations under /mnt/tank/configs/emby
- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.


## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Check the **Jellyfin Server Storage** box to **Enable Host Path for Jellyfin Server Config Volume** and select the correct path as described in the [Folder-Structure](/Folder-Structure) guide.
1. Click the **Add** button for **Jellyfin Server Extra Host Path Volumes** to pass the `media` directory into the container as **/media**.
1. Scroll to the **Resource Configuration** section and locate the checkbox for your GPU. If you do not see one, make sure to select the checkbox in **Apps → Configuration → Settings → Install NVIDIA Drivers**.

> Check out [the new docs](https://apps.truenas.com/resources/deploy-jellyfin) from TrueNAS
{.is-success}

> [Jellyfin.org official TrueNAS Documentation](https://jellyfin.org/docs/general/installation/advanced/truenas/)
{.is-success}


# 2 · Jellyfin Configuration

## 2.1 Setup Media Libraries

1. Click **\+ New Library**
2. Set the Content Type to match the folder (Movies for /media/movies and TV Shows for /media/tv)
1. Click on the **+** next to Folders and add the appropriate folder
1. Click the green **OK** at the bottom and leave the rest of the settings the same
1. Do the same thing for your second library for TV Shows.

## 2.2 Watching Content

To watch content on a device in your house on your home network, download the Jellyfin App from wherever your device gets apps from. When you start the Jellyfin app, manually add a server. Type in the IP Address of your server (for me it would look like `http://192.168.1.215`) and use the default port of `8096`. Once its connected you should see the sign-in screen with your name on it. If not, use the manual login and type the username and password you set up earlier.

# <img src="/youtube.png" class="tab-icon"> 3 · Video
https://youtu.be/VNhsR2lpntc

# 4 · Using NPM
If you are using [Nginx Proxy Manager](/nginx) to route your traffic use these settings to be sure you're not caching media:

![screenshot_from_2025-03-28_07-39-15.png](/screenshot_from_2025-03-28_07-39-15.png)

| Field | Value |
| --- | --- |
| location | `/` |
| Cog Icon | click this to expand the box at the bottom |
| scheme | http |
| Forward Hostname / IP | private IPv4 of your media server |
| Forward Port | port media server is running on | 
| expandable box | `add_header Cache-Control "no-store, no-cache, must-revalidate, proxy-revalidate"; add_header Pragma "no-cache"; add_header Expires "0";` | 

# 5 · Using an Intel iGPU

> This section is under development and may not work for you
{.is-warning}

1. Deploy Jellyfin using the above Docker Compose file, removing the lines with `nvidia` in them
1. In the TrueNAS shell, run the command `cd /dev/dri && ls -l`. You should see a line at the bottom that looks like:
```bash
crw-rw---- 1 root render 226, 128 Apr 15 14:44 renderD128
```
3. To find the ID of the `render` group, run the command `getent group render | cut -d: -f3`. In my case the output was `107`.
1. Now, add these lines into the Docker Compose file, using your values from above:
```yaml
group_add:
  - "107"
devices:
  - /dev/dri/:/dev/dri/
```
5. My final Docker Compose looks like:
```yaml
services:
  jellyfin:
    container_name: jellyfin
    image: lscr.io/linuxserver/jellyfin:latest
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    ports:
      - "8096:8096"
    restart: unless-stopped
    group_add:
      - "107"  # Access to /dev/dri/renderD128
      - "44"   # Access to /dev/dri/card0
    devices:
      - /dev/dri/:/dev/dri/
    volumes:
      - /mnt/tank/configs/jellyfin:/config
      - /mnt/tank/media:/media
```
6. If done correctly, you should see these two lines in your logs:
```bash
**** creating video group videol7cu with id 107 ****
**** adding /dev/dri/renderD128 to video group videol7cu with id 107 ****
**** adding /dev/dri/card0 to video group video with id 44 ****
```
7. Navigate to **Settings → Dashboard → Playback → Transcoding**
1. Change **Hardware Acceleration** to `Intel QuickSync (QSV)`
1. For the **QSV Device** add `/dev/dri/renderD128` if that was the output from Step 2
1. Follow the steps below to use AI to assist you in selecting the appropriate codecs for your iGPU
1. Remember to hit **Save** at the bottom when you're done

# 6 · Transcoding settings
Many people ask how to setup the transcoding section of Jellyfin for their specific GPU. I usually tell them "it depends" because each GPU has different capabilities. However, I will tell you a great way to answer this for your individual GPU is ChatGPT. To do this, take a screenshot of the transcode page and ask: `"if i have <insert_GPU> which of these should i select for hardware decoding and tone mapping?"`

![screenshot_from_2025-04-08_18-43-41.png](/screenshot_from_2025-04-08_18-43-41.png)


ChatGPT can read the image and tell me which boxes to check and what my Tone Mapping settings should be.

As an example, here was the response:

> With an NVIDIA Quadro P2000 GPU, you can enable several hardware decoding options thanks to its support for NVDEC (NVIDIA Video Decoder). The P2000 supports hardware decoding for a wide variety of codecs, so here's what you should ideally enable in Jellyfin based on your screenshot:
> ✅ Safe to Enable with P2000:
> 
> H264 ✅ (already enabled)
> 
> HEVC ✅ (Main/8-bit only; 10-bit HEVC decoding is supported via partial acceleration)
> 
> MPEG2 ✅
> 
> MPEG4 ✅
> 
> VC1 ✅ (already enabled)
> 
> VP8 ✅
> 
> VP9 ✅ (8-bit; 10-bit is partially supported)
> 
> HEVC 10bit ✅ (Partial decoding support—still useful)
> 
> VP9 10bit ✅ (already enabled; partial support)
> 
> ❌ Not Supported or Questionable on P2000:
> 
> AV1 ❌ (AV1 is not supported on P2000)
> 
> HEVC RExt 8/10bit ❌ (Refers to extended profiles—unsupported)
> 
> HEVC RExt 12bit ❌
> 
> Recommended settings for you:
> 
> ✅ Enable: H264, HEVC, MPEG2, MPEG4, VC1, VP8, VP9, HEVC 10bit, VP9 10bit
> 
> ❌ Disable or leave unchecked: AV1, HEVC RExt 8/10bit, HEVC RExt 12bit
> 
> Bonus Tips:
> 
> Enhanced NVDEC Decoder ✅ (keep this enabled — improves decoding on newer drivers)
> 
> Enable Hardware Encoding ✅ (already enabled — the P2000 supports NVENC)
> 
> For encoding, H264 is your best option. You can enable HEVC encoding if you want smaller file sizes and are okay with slightly slower performance.
> 
> ✅ Recommended Tone Mapping Settings for P2000:
> 
> Enable Tone Mapping: ✅ Yes (only if you're playing HDR content on SDR displays).
> 
> Tone Mapping Algorithm: BT.2390 (recommended default — a perceptually better curve for HDR→SDR conversion).
> 
> Tone Mapping Mode: Auto (you can try RGB if you notice washed-out colors or blown highlights).
> 
> Tone Mapping Range: Auto (keeps the same color range as input; usually best unless you know you're targeting limited-range SDR).

# <img src="/youtube.png" class="tab-icon"> 7 · Plug-ins

https://youtu.be/6oUOi8-09E8