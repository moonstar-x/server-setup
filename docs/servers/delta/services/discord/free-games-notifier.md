# Free Games Notifier

[discord-free-games-notifier](https://github.com/moonstar-x/discord-free-games-notifier) is a bot that notifies a channel whenever there's a free game on Steam or Epic Games.

There is an official image for this service that we'll use: [ghcr.io/moonstar-x/discord-free-games-notifier](https://github.com/moonstar-x/discord-free-games-notifier).

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
    image: ghcr.io/moonstar-x/discord-free-games-notifier:latest
    restart: unless-stopped
    depends_on:
      - redis
      - postgres
    environment:
      TZ: America/Guayaquil
      DISCORD_TOKEN: DISCORD_TOKEN_HERE
      DISCORD_SHARING_ENABLED: false
      DISCORD_SHADING_COUNT: auto
      DISCORD_PRESENCE_INTERVAL: 300000
      REDIS_URI: redis://redis:6379
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_DATABASE: free-games
      POSTGRES_USER: DATABASE_USER
      POSTGRES_PASSWORD: DATABASE_PASSWORD

  crawler:
    image: ghcr.io/moonstar-x/free-games-crawler:latest
    restart: unless-stopped
    depends_on:
      - redis
    environment:
      TZ: America/Guayaquil
      REDIS_URI: redis://redis:6379

  redis:
    image: redis:7.4-rc2-alpine
    restart: unless-stopped
    command: redis-server --save 60 1 --loglevel warning
    volumes:
      - ./crawler-data:/data

  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: free-games
      POSTGRES_USER: DATABASE_USER
      POSTGRES_PASSWORD: DATABASE_PASSWORD
```

!!! note
    Make sure to change `DISCORD_TOKEN_HERE` to your bot's Discord token.

!!! note
    Make sure to change `DATABASE_USER` and `DATABASE_PASSWORD` to a custom secret value.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
