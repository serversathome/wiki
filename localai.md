---
title: AI
description: How to deploy local AI containers on TrueNAS
published: true
date: 2025-07-09T13:14:12.614Z
tags: 
editor: markdown
dateCreated: 2025-02-04T22:59:57.204Z
---

# ![](/ollama.png){class="tab-icon"} What is Ollama?
**Ollama** - Get up and running with large language models. Get up and running with Llama 3.2, Mistral, Gemma 2, and other large language models.

# ![](/open-webui.png){class="tab-icon"} What is Open WebUI?
**Open WebUI** - User-friendly AI Interface (Supports Ollama, OpenAI API, ...) Open WebUI is an extensible, feature-rich, and user-friendly self-hosted WebUI designed to operate entirely offline. It supports various LLM runners, including Ollama and OpenAI-compatible APIs.

# <img src="/truenas.png" class="tab-icon"> 1 · Deploy Containers

![](/screenshot_from_2025-02-06_07-32-03.png)

For both apps we want to create a dataset for each using the **apps** Dataset Preset for permissions. 
> Make sure the dataset for **Ollama** is in a pool with some space because that is where we will be storing all of the AI models, some of which can be large
{.is-info}


The changes we need to make on the **Ollama** configuration are:

1.  set the **Ollama Data Storage** to host path
2.  Increase resource limits to whatever we can spare
3.  passthrough the GPU

![](/screenshot_from_2025-02-06_07-38-46.png)

The changes we need to make on the **Open WebUI** configuration are:

1.  set the **Open WebUI Data Storage** to host path
2.  set the **Open WebUI Ollama Storage** to hostpath pointed to the *ollama* dataset we created earlier

# Ollama ROCM
**Run Ollama with an unofficialy supported AMD GPU**

The following will show an example compose file that should get most users up and running with an AMD GPU, specefically those that are trying to use a GPU that is not officially supported by Ollama but can utilize ROCM. Make sure to change the path under volumes to match your setup.

HSA_OVERRIDE_GFX_VERSION will depend on your GPU model: You can reference the following site - https://rocm.docs.amd.com/en/latest/reference/gpu-arch-specs.html and plug in the LLVM target name to HSA_OVERRIDE_GFX_VERSION under the **environment** section below. For example if you are using RDNA 2, stick with 10.3.0. Rule of thumb, ignore the sub version number, i.e. the 2 in gfx1032.

**Compose Example**
```
services:
  ollama:
    cap_drop:
      - ALL
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4096M
    devices:
      - /dev/dri:/dev/dri
      - /dev/kfd:/dev/kfd
    environment:
      HSA_OVERRIDE_GFX_VERSION: 10.3.0
      OLLAMA_HOST: 0.0.0.0:30068
      TZ: America/New_York
      UMASK: '002'
      UMASK_SET: '002'
    group_add:
      - 44
      - 107
      - 568
    healthcheck:
      interval: 10s
      retries: 30
      start_period: 10s
      test: timeout 1 bash -c 'cat < /dev/null > /dev/tcp/127.0.0.1/30068'
      timeout: 5s
    image: ollama/ollama:rocm
    platform: linux/amd64
    ports:
      - mode: ingress
        protocol: tcp
        published: 30068
        target: 30068
    privileged: False
    restart: unless-stopped
    security_opt:
      - no-new-privileges=true
    stdin_open: False
    tty: False
    volumes:
      - /mnt/name_of_your_pool/name_of_your_dataset:/root/.ollama
```      
Once you have successfully installed the container, you should be able to open the logs and see the following or similar message that will verify ollama is able to see and use your AMD GPU. 

![Screenshot_2025-07-17-113913.png](/Screenshot_2025-07-17-113913.png)

After you have confirmed your GPU is being utilized, you can follow the video below to pull LLMs into ollama.

Keep in mind initial response by ollama will be slow, the GPU needs to "wake up" so to speak. After about 3 minutes the LLM will become idle and your GPU will "go to sleep". This is not inherently a bad thing, as this keeps system resources, your GPU, from running persistently.

Here is an example of the time and token response of an RX6650 XT 8GiB:
This is using gemma3:4b and asking the question, "What is the secret of the universe?"

![Screenshot_2025-07-17-115222.png](/Screenshot_2025-07-17-115222.png)

I must also give credit to two specific posts that made this all possible.
1. https://forums.truenas.com/t/amd-gpu-configuration-for-open-web-ui/43812/4
2. https://major.io/p/ollama-with-amd-radeon-6600xt/


# 2 · Open WebUI Configuration

1. After creating a username and password, click the colored circle icon in the very top right corner and then select **Admin Panel**
1. Navigate to the **Settings** tab and then **Connections**
1. Change the section at the bottom labeled **Manage Ollama API Connections** and click the **cog icon** at the end of the row
1. Change the URL to the IP and Port that Ollama is running on. Click **Save**.
1. Staying in the **Settings** tab, Navigate to the **Models** (right below **Connections**) and in the top right corner click the **download icon** (alt text is *manage models*). 
1. You should see the correct Ollama IP and port in the top box. Enter the model name in the second box labeled **Pull a model from Ollama.com**. 
> If you do not know the model name and size, click the “click here” hyperlink right below to be taken to [https://ollama.com/library](https://ollama.com/library) to search for available models
{.is-info}

7. Once you enter a model name, click the download icon at the end of the box to download the model. It will take awhile to download, check the SHA, and create the manifest, but once completed you can now start a new chat with the new model. 
> 
> To allow all users to see the model, click the **Pencil icon** at the end of the row and change the **Visibility** to **Public**.
{.is-warning}


# <img src="/youtube.png" class="tab-icon"> 3 ·Video Walkthrough

[https://youtu.be/b2HeHDUbkec](https://youtu.be/b2HeHDUbkec)