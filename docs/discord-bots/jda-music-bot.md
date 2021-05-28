# JDA Music Bot

[jagrosh's MusicBot](https://github.com/jagrosh/MusicBot) is a very powerful music bot written in Java with JDA. This bot does not have an official *Docker* image so we'll create a `Dockerfile` for this one.

## Pre-Installation

We'll create a folder in the main user's home where all the bot's data will be saved.

```bash
mkdir ~/discord/jda-music-bot
```

We'll also need to download the latest version of the jar from [here](https://github.com/jagrosh/MusicBot/releases) which will be downloaded inside an `app` folder where all the bot's data will be stored.

```bash
cd ~/discord/jda-music-bot && mkdir app && cd app
wget https://github.com/jagrosh/MusicBot/releases/download/0.3.4/JMusicBot-0.3.4.jar
mv JMusicBot-0.3.4.jar JMusicBot.jar
```

Now, we should have a `JMusicBot.jar` file inside of `~/discord/jda-music-bot/app`.

## Configuration

This bot requires its configuration to be saved in a text file. We'll create a file called `config.txt` with:

```bash
nano ~/discord/jda-music-bot/app/config.txt
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

## Dockerfile

Since the bot does not have an official *Docker* image, we'll create a `Dockerfile` to run any java `jar` file through volumes. The content of the `Dockerfile` file is as follows:

```docker
FROM openjdk:8

WORKDIR /opt/app
VOLUME /opt/app

CMD ["java", "-Dnogui=true", "-jar", "JMusicBot.jar"]
```

## Docker Compose

The bot will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  bot:
    build: .
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./app:/opt/app
```

Before starting, we need to build this image, do so with:

```bash
docker-compose build
```

## Running

Start up the bot with:

```bash
docker-compose up -d
```

That's it! The bot will auto-start on system startup and restart on failure.
