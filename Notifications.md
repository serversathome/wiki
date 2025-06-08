---
title: Notifications
description: Learn how to setup notifications for all of your services!
published: true
date: 2025-06-08T18:38:58.835Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:28:42.117Z
---

# Gotify

> I am moving away from Discord in favor of [Gotify](/gotify). Unfortunately, I'm told there is no mobile app for iOS. Even though there is a Gotify template for Unpackerr, I could not get it working. Other than that, I like it much more.
{.is-info}


# Discord

There are so many ways to get notifications, there could be an entire wiki dedicated to that alone. For simplicity's sake, we are going to go with Discord here. The prerequisite here is that you have signed up for a Discord account.

## Setting Up A Server

Start by launching Discord and set up a private server, calling it whatever you want. Follow the steps:

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-28-15.png)

1. Click “Create My Own”

1. Add a **Text Channel**. Slide the slider to *Private Channel*. Skip the step when it asks you to add moderators.

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-31-13.png)

3. Click the cog icon next to the channel name and then click the **Integrations** tab in the left pane. Click **Create Webhook**, the expand the Spidey Bot box, and **Copy the Webhook URL**.

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-34-09.png)

4. Paste that to a text doc somewhere because we will need it later. 

# TrueNAS Scale

Now that we have a Discord server to push to, let's enable TrueNAS to notify us of alerts.

5. Click on the Alert Bell in the top right corner, then the Cog icon to bring us to **Alert Settings**. 

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-37-33.png)

6. Add an alert using the blue button
7. Add the description and choose **Slack** for the *Type*
8. Paste the webhook into the bottom box and add “/slack” to the end with no spaces
9. Send the Test Alert to make sure you did it correctly.

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-38-44.png)

# Radarr/Sonarr/Prowlarr

Since all the \*arrs are basically the same UI, we will follow the same steps for all of them.

1. Navigate to **Settings** > **Connect** (on Prowlarr its **Settings** > **Notifications**) 
1. Click the **\+** box. Select *Discord*. 

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-44-43.png)

3. Paste the webhook to the **Webhook URL** box
> I suggest you deselect some of the check boxes though, otherwise you will be getting alerts all the time. Usually I just have grab, import, added, health issue, and manual interaction to keep the noise down.
{.is-info}


# Unpackerr

Since this is command line, it will be a little uglier, but no less difficult. 

If you are using Docker, you will see the section for Webhook in the config file. These are the values which need to be modified:

| Config Name | Variable Name | Default / Note |
| --- | --- | --- |
| webhook.url | `UN_WEBHOOK_0_URL` | paste your Discord webhook URL here |
| webhook.name | `UN_WEBHOOK_0_NAME` | call this Discord |
| webhook.nickname | `UN_WEBHOOK_0_NICKNAME` | leave default |
| webhook.channel | `UN_WEBHOOK_0_CHANNEL` | leave default |
| webhook.timeout | `UN_WEBHOOK_0_TIMEOUT` | leave default |
| webhook.silent | `UN_WEBHOOK_0_SILENT` | leave default |
| webhook.ignore\_ssl | `UN_WEBHOOK_0_IGNORE_SSL` | leave default |
| webhook.exclude | `UN_WEBHOOK_0_EXCLUDE` | leave default |
| webhook.events | `UN_WEBHOOK_0_EVENTS` | leave default |
| webhook.template\_path | `UN_WEBHOOK_0_TEMPLATE_PATH` | leave default |
| webhook.template | `UN_WEBHOOK_0_TEMPLATE` | use “discord” |
| webhook.content\_type | `UN_WEBHOOK_0_CONTENT_TYPE` | leave default |

In TrueNAS, we only need to add 3 values in the *Extra Environment Variables* for Unpackerr:

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-54-02.png)

| Name | Value |
| --- | --- |
| UN_WEBHOOK_0_URL  | enter webhook here |
| UN_WEBHOOK_0_NAME | Unpackerr/Discord |
|  UN_WEBHOOK_0_TEMPLATE |  discord |


# Jellyseerr

1. Navigate to **Settings** > **Notifications** > **Discord** 
1. Paste your webhook in the correct box.

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_17-56-50.png)

# Emby

> If you do not have the Premium version of Emby, this will not work because it requires an add-on
{.is-danger}

1. Navigate to **Settings** > **Advanced** > **Plugins**
1. Click the tab at the top to select **Catalog**
1. Scroll down to the *Notifications* section and look for the Discord option.

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_18-04-23.png)

4. Once it is installed and you have rebooted your server, navigate to **Settings** > **Notifications** 
5. Click **Add Notification**. Select Discord and enter the following values:

|   Field  | Value |
| --- | --- |
| Name | Emby |
| LabelDiscordWebhookUrl | paste webhook here |
| LabelWebhookUsername | Emby |
| Events | select the events you want to be notified for |
| External | select the last 2 checkboxes for External \| External Notification via Emby Server API |

6. Click **Save** 

# Uptime Kuma

1. Navigate to **Settings** > **Notifications** > **Setup** **Notification**
1. Paste your webhook URL and give it a friendly name. 
1. Enable the two sliders at the bottom to enable the notification by default and apply it to existing monitors.

![](https://wiki.hydrology.cc/screenshot_from_2024-02-12_18-36-08.png)