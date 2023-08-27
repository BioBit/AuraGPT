# AuraGPT

> ### A ChatGPT Discord Bot for [Neural Nexus](https://portal.neuralnexus.ch/)

---
> **Warning**
>
> #### Modified Bot based on: https://github.com/Zero6992/chatGPT-discord-bot
> #### This Bot requires OpenAI API access. [Every prompt will cost a few cents](https://openai.com/pricing).

### Chat

![image](https://user-images.githubusercontent.com/89479282/206497774-47d960cd-1aeb-4fba-9af5-1f9d6ff41f00.gif)

To continue the conversation -> reply or start a thread. The continue the conversation using the `/chat` command.

## Setup in Portainer

Note: These instructions are for a deployment using [Portainer](https://www.portainer.io/).

## Critical prerequisites to install

Clone this repository into a directory on your host machine. The container will need to access certain source fails when it is composing.

---
## Step 1: Create a Discord bot

1. Go to https://discord.com/developers/applications create an application
3. Build a Discord bot under the application.
> General information: AuraGPT, Description (Neural Nexus AI, powered by ChatGPT)

> Bot: Give it a name that you like, for example `Nexus Mind`. Make sure to disable 'Public Bot' option. Enable 'Message Content Intent'
4. Get the token from bot setting

   ![image](https://user-images.githubusercontent.com/89479282/205949161-4b508c6d-19a7-49b6-b8ed-7525ddbef430.png)
6. Store the token to `.env` under the `DISCORD_BOT_TOKEN`

   <img height="190" width="390" alt="image" src="https://user-images.githubusercontent.com/89479282/222661803-a7537ca7-88ae-4e66-9bec-384f3e83e6bd.png">

7. Turn MESSAGE CONTENT INTENT `ON`

   ![image](https://user-images.githubusercontent.com/89479282/205949323-4354bd7d-9bb9-4f4b-a87e-deb9933a89b5.png)

8. Invite your bot to your server via OAuth2 URL Generator

   ![image](https://user-images.githubusercontent.com/89479282/205949600-0c7ddb40-7e82-47a0-b59a-b089f929d177.png)

9. Under 'Scopes' select 'bot' > 'Bot Permissions' will appear
10. Under 'Bot Permissions' enable all 'text permissions' > copy the generated URL at the bottomt of the page.
11. In a new browser tab or window > paste the copied. If you are the admin of a server, you should receive the option to invite the bot to your server.
---
> **Note**
>
> In Step 2, you only need to complete the authentication process for the model you want to use (it's not necessary to complete all Step 2)
>
> Remember to modify `CHAT_MODEL` to the default model you want to use in `.env` file

## Step 2: Official API authentication

### Geanerate an OpenAI API key
1. Go to https://beta.openai.com/account/api-keys

2. Click Create new secret key

   ![image](https://user-images.githubusercontent.com/89479282/207970699-2e0cb671-8636-4e27-b1f3-b75d6db9b57e.PNG)

3. Store the SECRET KEY to `.env` under the `OPENAI_API_KEY`

---
## Step 3: Run the bot with Portainer

### Build the Docker image & Run the Docker container

In Portainer, go to your Environment > Image > '+Build a new image'

> name: auragpt

> Build method: Web editor

Copy&Paste the following (also available in the 'Dockerfile')

```
FROM python:3.10-bullseye

ENV PYTHONUNBUFFERED 1
RUN apt-get update && apt-get install -y \
  build-essential \
  libcairo2-dev \
  cargo \
  libfreetype6-dev \
  gcc \
  libgdk-pixbuf2.0-dev \
  gettext \
  libjpeg-dev \
  liblcms2-dev \
  libffi-dev \
  musl-dev \
  libopenjp2-7-dev \
  libssl-dev \
  libpango1.0-dev \
  poppler-utils \
  postgresql-client \
  libpq-dev \
  python3-dev \
  rustc \
  tcl-dev \
  libtiff5-dev \
  tk-dev \
  zlib1g-dev

RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
ENV PATH="/root/.cargo/bin:${PATH}"
RUN rustup update

RUN pip3 install cryptography
COPY ./ /DiscordBot
WORKDIR /DiscordBot
RUN pip3 install -r requirements.txt

CMD ["python3", "main.py"]
```

> In 'Upload', make sure to upload the files `requirements.txt` and `main.py`.

> Click 'Build the image' at the bottom. This will take several minutes and the image will be approx. 3 GB large.

### Create the Portainer stack that will run the container

In Portainer, go to your Environment > Stacks > '+Add stack'

> name: auragpt

> Build method: Web editor

Copy&Paste the following (also available in the 'docker-compose.yml')

```
version: "3"

services:
  chatgpt-discord-bot:
    build: .
    image: auragpt:latest
    container_name: auragpt
    env_file:
      - stack.env
    restart: unless-stopped
    volumes:
      - type: bind
        source: /opt/portainer/portainer_data/compose/AuraGPT
        target: /DiscordBot
 
volumes:
  chatgpt-discord-bot:
```
> **Note**
>
> The volume 'source:' will be on your host machine. You can provide any folder that you like. Preferably, it is in your home directory.


### Have a good chat!
---

## Optional: Setup system prompt

* A system prompt would be invoked when the bot is first started or reset
* You can set it up by modifying the content in `system_prompt.txt`
* All the text in the file will be fired as a prompt to the bot
* Get the first message from ChatGPT in your discord channel!
* Go Discord setting turn `developer mode` on

   1. Right-click the channel you want to recieve the message, `Copy  ID`

        ![channel-id](https://user-images.githubusercontent.com/89479282/207697217-e03357b3-3b3d-44d0-b880-163217ed4a49.PNG)

   2. For Neural Nexus, the channel is #ai-introduction (1116284755533639691)
   3. paste it into `.env` under `DISCORD_CHANNEL_ID`

## Optional: Disable logging

* Set the value of `LOGGING` in the `stack.env` to False

> **Note**
>
> Logging is disabled by default.

------
## Commands

* `/chat [message]` Chat with ChatGPT!
* `/draw [prompt]` Generate an image with the Dalle2 model
* `/switchpersona [persona]` Switch between optional chatGPT personalities.
   * `Nexus Mind`: Default personality implant for Neural Nexus
   * `random`: Picks a random persona
   * `chatGPT`: Standard chatGPT mode
   * `dan`: Dan Mode 11.0, infamous Do Anything Now Mode
   * `sda`: Superior DAN has even more freedom in DAN Mode
   * `confidant`: Evil Confidant, evil trusted confidant
   * `based`: BasedGPT v2, sexy gpt
   * `oppo`: OPPO says exact opposite of what chatGPT would say
   * `dev`: Developer Mode, v2 Developer mode enabled

* `/private` ChatGPT switch to private mode (disabled)
* `/public` ChatGPT switch to public mode
* `/replyall` ChatGPT switch between replyAll mode and default mode (disabled)
* `/reset` Clear ChatGPT conversation history
* `/chat-model` Switch different chat model
   * `OFFICIAL-GPT-3.5`: GPT-3.5 model
   * `OFFICIAL-GPT-4.0`: GPT-4.0 model (make sure your account can access gpt-4 model)

### Special Features

#### Draw (disabled)

![image](https://user-images.githubusercontent.com/91911303/223772051-13f840d5-99ef-4762-98d2-d15ce23cbbd5.png)

#### Switch Persona

> **Warning**
>
> Using certain personas may generate vulgar or disturbing content. Use at your own risk.

![image](https://user-images.githubusercontent.com/91911303/223772334-7aece61f-ead7-4119-bcd4-7274979c4702.png)


#### Mode

* `public mode (default)`  the bot directly reply on the channel

  ![image](https://user-images.githubusercontent.com/89479282/206565977-d7c5d405-fdb4-4202-bbdd-715b7c8e8415.gif)

* `private mode` the bot's reply can only be seen by the person who used the command (disabled)

  ![image](https://user-images.githubusercontent.com/89479282/206565873-b181e600-e793-4a94-a978-47f806b986da.gif)

* `replyall mode` the bot will reply to all messages in the channel without using slash commands (`/chat` will also be unavailable) (disabled)

   > **Warning**
   > The bot will easily be triggered in `replyall` mode, which could cause program failures
 ---
