# Free Games Notifier

[discord-free-games-notifier](https://github.com/moonstar-x/discord-free-games-notifier) is a bot that notifies a channel whenever there's a free game on Steam or Epic Games.

There is an official image for this service that we'll use: [moonstarx/discord-free-games-notifier](https://hub.docker.com/r/moonstarx/discord-free-games-notifier).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/discord/discord-free-games-notifier
```

## Docker Compose

*Free Games Notifier* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

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
```

!!! note
    Make sure to change `DISCORD_TOKEN_HERE` to your bot's Discord token.

!!! note
    Make sure to change `OWNER_ID_HERE` to your Discord user's ID.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
