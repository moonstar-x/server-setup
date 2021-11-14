# Music 24/7 Bot

[discord-music-24-7](https://github.com/moonstar-x/discord-music-24-7) is a music bot with auto-pause capabilities that pauses the music playback when nobody is in the channel. It can play music from YouTube, SoundCloud and local files. This bot has an image available on [Docker Hub](https://hub.docker.com/r/moonstarx/discord-music-24-7) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the bot's data will be saved.

```bash
mkdir ~/discord/discord-music-24-7
```

## Docker Compose

The bot will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  bot:
    image: moonstarx/discord-music-24-7:latest
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./data:/opt/app/data
    environment:
      - DISCORD_TOKEN=<DISCORD_TOKEN_HERE>
      - DISCORD_PREFIX=s!
      - DISCORD_CHANNEL_ID=<CHANNEL_ID_HERE>
      - DISCORD_OWNER_ID=<OWNER_ID_HERE>
      - TZ=America/Guayaquil
```

## Running

Start up the bot with:

```bash
docker-compose up -d
```

That's it! The bot will auto-start on system startup and restart on failure.
