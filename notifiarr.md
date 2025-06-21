---
title: Notifiarr
description: A fantastic tool to add in with your ArrStack. Simplifies space and file types. Also gives well notifications.
published: true
date: 2025-06-20T20:24:47.088Z
tags: 
editor: markdown
dateCreated: 2025-06-20T20:23:44.026Z
---

![image](https://github.com/user-attachments/assets/bf101ffd-2fbf-4f92-8724-76a37afd6092)

# What is Notifiarr
To put it simply Notifiarr is a app that takes Trashguides (what I use it for) and helps with making profiles or formats in your ArrStack. It has plentiful of features that you can delve into. But today I will be showing you how to set it up for Trashguides and your ArrStack.

# Setup of Notifiarr
<i>For this setup I am using TrueNAS Fangtooth Community Edition 25.04</i>
<br><br>This will be a lot of back and forth<br>

First off go to <a hfref="https://notifiarr.com/guest/register">Notfiarr</a> and create a account.

<img src="https://github.com/user-attachments/assets/3e24b851-ff45-488d-8c5d-e9286592f198" width="500" height="400">



# Next it will ask you to login to Discord 
This is used to setup notifications. What I did is just create a new server that is private for my eyes only. You do not need anything to go to the server that you do not want. It is a very powerful integration.
You will need to also create a bot for Discord to integrate it. 

### Use this page for help on creating a bot: <a href="https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server">Information how to create a bot,</a> if you do not know how to.

<img src="https://github.com/user-attachments/assets/54256674-fca4-4ef8-949c-846d7d9acad6" width="600" height="300">

Follow the instructions after you connect your Discord and Notfiarr. Once done creating the Bot and connecting it. Just go through the first setup.
It should bring you to your profile page.

<img src="https://github.com/user-attachments/assets/a56f1cb3-2922-4b78-8a66-fbf0da873db0" width="600" height="300">


## Time to go over to TrueNAS and search up Notifiarr in the app store.

<img src="https://github.com/user-attachments/assets/7a47eb6d-84c7-4467-9e8f-d71b093dd2ae" width="500" height="200">


## In order to get your API key head back to the notifiarr website in your profile. Scroll down till you see this section and copy your api key. 
<img src="https://github.com/user-attachments/assets/9563760b-d89c-495a-b06a-87d730c564f9" width="500" height="200">

<br>
These are the only things you will need to change.<br>
<img src="https://github.com/user-attachments/assets/a023a023-29d7-4eaf-9124-d11ea94a4348" width="300" height="200">
<img src="https://github.com/user-attachments/assets/0065d224-3647-4baf-be13-4b58c584f7be" width="300" height="300"><br>

Once it is installed go to your logs and grab your password. It is randomly generated. Suggested to change your password once setup.<br>

Login to your instance.<br>
<img src="https://github.com/user-attachments/assets/285b21c8-02eb-480d-9b6b-cfea8c53830e" width="500" height="400"><br>

On notifiarr website go to setup on the left hand side and select integrations. Then client settings. Once it is open scroll down to see a green connect symbol. or a popup in the bottom right corner will say it is connected.

<img src="https://github.com/user-attachments/assets/99acfedb-adbc-4a42-be0c-b2633d1aff76" width="200" height="400">
<img src="https://github.com/user-attachments/assets/c929f609-6822-4ca7-8454-683b3d3982fb" width="500" height="400">

<br>*Popup in Bottom Right*<br>![image](https://github.com/user-attachments/assets/f3b8f83f-09bc-4a87-a040-6ece04ff1a91)
<br>*Green connect status*<br>
![image](https://github.com/user-attachments/assets/8c672f5c-b198-4b62-8e02-384161778e9c)<br>

Once they are connected we are going to go back to your notifiarr instance to setup a Arr instance.<br>
![image](https://github.com/user-attachments/assets/77fec8b4-b4f2-4e38-9596-5272bc633d9f)<br>
*click the plus sign and put in your instance name, url, username and password*<br>
![image](https://github.com/user-attachments/assets/cf606749-e09f-40c5-83c2-bf27905f2326)
*grab your api key from your Arr instance*<br>
![image](https://github.com/user-attachments/assets/e4f40f8d-feda-44ba-a801-d00c168ea01d)<br>
*Go back to radarr and click on connect once you have clicked save*<br>
<img src="https://github.com/user-attachments/assets/92e0f08c-463b-4645-ae7a-6310160068ae" width="200" height="400"><br>
We are going to add notifiarr connection to radarr now. In order to get your api key for the connection go to the notifiarr website. Add a Integration and select radarr. Then go to your profile page and scroll back down and create a API key for radarr
![image](https://github.com/user-attachments/assets/8d3e2b85-0c0e-4e31-b856-545504b4e49b)
For the name of the Connection in radarr make sure to add a character at the end of it, any other arr's you might do will need a different character from this one as well. you you connect them go back to notifiarr website and open the radarr itegration.<br><br>
*Click this button*
![image](https://github.com/user-attachments/assets/47ccce49-a2da-4bb8-9dde-add97e1168df)

Back in Radarr once succesful you should see two connections now.

![image](https://github.com/user-attachments/assets/5b04bb4e-b756-4a6c-9a51-0727729429fe)

We are now done with the connections. Everything else will be handled in Notifiarr.

## Add integration of Trash Guides. 
Once added you will set it up <br>
![image](https://github.com/user-attachments/assets/77757789-edd9-4eed-8bfe-6777866a3780)<br>
Click on profiles. Once up you will click on Add new / Link existing
![image](https://github.com/user-attachments/assets/3d351c2e-93e4-45a9-b430-abb7957b742a)<br>

The one I personally use is SQP-1 (2160P) 
![image](https://github.com/user-attachments/assets/9288825e-729a-4fc6-a3ea-09d31ec6b1ea)

Select it and it will load the config. I would not mess with anything unless you know how to work trash guides. Just Scroll down and click Save Profile and Sync
![image](https://github.com/user-attachments/assets/a8c83bb4-a459-46cf-abbd-e2cd2a086589)


On the next page click Sync Profiles
![image](https://github.com/user-attachments/assets/af9b9bbe-b437-4840-a2ae-18567ae6195b)

That is it. It might take a couple of minutes to get it imported into your arr instance.

![image](https://github.com/user-attachments/assets/d0e0454f-ef70-419e-89cc-05619c26ee03)

Once it is however you just select that when doing any of your arr work in radarr.




