# Music 24/7

[discord-music-24-7](https://github.com/moonstar-x/discord-music-24-7) is a music bot with auto-pause capabilities that pauses the music playback when nobody is in the channel. It can play music from YouTube, SoundCloud and local files.

There is an official image for this service that we'll use: [moonstarx/discord-music-24-7](https://hub.docker.com/r/moonstarx/discord-music-24-7).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/discord/discord-music-24-7
```

## Docker Compose

*Music 24/7* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  bot:
    image: moonstarx/discord-free-games-notifier:latest
    restart: unless-stopped
    volumes:
      - ./data:/opt/app/data
    environment:
      TZ: America/Guayaquil
      DISCORD_TOKEN: DISCORD_TOKEN_HERE
      DISCORD_PREFIX: n!
      DISCORD_OWNER_ID: OWNER_ID_HERE
      DISCORD_CHANNEL_ID: CHANNEL_ID_HERE
      DISCORD_PAUSE_ON_EMPTY: false
```

!!! note
    Make sure to change `DISCORD_TOKEN_HERE` to your bot's Discord token.

!!! note
    Make sure to change `OWNER_ID_HERE` to your Discord user's ID.

!!! note
    Make sure to change `CHANNEL_ID_HERE` to your voice channel's ID.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
