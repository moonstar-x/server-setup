# JDA Music Bot

[jagrosh's MusicBot](https://github.com/jagrosh/MusicBot) is a very powerful music bot written in Java with JDA.

There is no official image for this service, so we'll use [openjdk](https://hub.docker.com/_/openjdk) to run this service manually.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/discord/discord-jda-music-bot
```

We'll also need to download the latest version of the jar from [here](https://github.com/jagrosh/MusicBot/releases) which will be downloaded inside an `app` folder where all the bot's data will be stored.

```bash
cd ~/services/discord/discord-jda-music-bot && mkdir app && cd app
wget https://github.com/jagrosh/MusicBot/releases/download/0.4.0/JMusicBot-0.4.0.jar
mv JMusicBot-0.4.0.jar JMusicBot.jar
```

Now, we should have a `JMusicBot.jar` file inside of `~/services/discord/discord-jda-music-bot/app`.

## Configuration

This bot requires its configuration to be saved in a text file. We'll create a file called `config.txt` with:

```bash
nano ~/services/discord/discord-jda-music-bot/app/config.txt
```

And its content should be as follows:

```text
token=<DISCORD_TOKEN_HERE>
owner=<OWNER_ID_HERE>
prefix=%
game="DEFAULT"
status=ONLINE
songinstatus=true
altprefix="NONE"
stayinchannel=true
maxtime=0
updatealerts=false
```

!!! note
    Make sure to change `OWNER_ID_HERE` to your Discord user's ID.

!!! note
    Make sure to change `DISCORD_TOKEN_HERE` to your bot's Discord token.

## Docker Compose

*JDA Music Bot* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  bot:
    image: openjdk:23-slim
    restart: unless-stopped
    volumes:
      - ./app:/opt/app
    working_dir: /opt/app
    command: ['java', '-Dnogui=true', '-jar', 'JMusicBot.jar']
    environment:
      TZ: America/Guayaquil
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
