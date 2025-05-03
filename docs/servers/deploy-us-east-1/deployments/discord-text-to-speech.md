# Text-to-Speech for Discord

[discord-tts-bot](https://github.com/moonstar-x/discord-tts-bot) is a bot that uses the Google Translate API to utter the messages you send to the bot in any language.

There is an official image for this service that we'll use: [moonstarx/discord-tts-bot](https://hub.docker.com/r/moonstarx/discord-tts-bot).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/deployments/discord-tts-bot
```

## Docker Compose

*Text-to-Speech* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  bot:
    image: moonstarx/discord-tts-bot:latest
    restart: unless-stopped
    depends_on:
      - redis
    volumes:
      - ./data:/opt/app/data
    environment:
      TZ: America/Guayaquil
      DISCORD_TOKEN: ${DISCORD_TOKEN}
      DISCORD_PREFIX: $$
      DISCORD_OWNER_ID: ${DISCORD_OWNER_ID}
      DISCORD_DEFAULT_DISCONNECT_TIMEOUT: 10
      DISCORD_PROVIDER_TYPE: redis
      DISCORD_REDIS_URL: redis://redis:6379

  redis:
    image: redis:latest
    restart: unless-stopped
    volumes:
      - ./data:/data
    command: redis-server --save 60 1 --loglevel warning
    environment:
      TZ: America/Guayaquil

```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DISCORD_TOKEN=
DISCORD_OWNER_ID=
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
