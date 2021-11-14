# Free Games Notifier

[discord-free-games-notifier](https://github.com/moonstar-x/discord-free-games-notifier) is a bot that notifies a channel whenever there's a free game on Steam or Epic Games. This bot has an image available on [Docker Hub](https://hub.docker.com/r/moonstarx/discord-free-games-notifier) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the bot's data will be saved.

```bash
mkdir ~/discord/discord-free-games-notifier
```

## Docker Compose

The bot will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  bot:
    image: moonstarx/discord-free-games-notifier:latest
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./data:/opt/app/data
    environment:
      - DISCORD_TOKEN=<DISCORD_TOKEN_HERE>
      - DISCORD_PREFIX=!
      - DISCORD_OWNER_ID=<OWNER_ID_HERE>
      - TZ=America/Guayaquil
```

## Running

Start up the bot with:

```bash
docker-compose up -d
```

That's it! The bot will auto-start on system startup and restart on failure.
