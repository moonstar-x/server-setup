# Text-to-Speech Bot

[discord-tts-bot](https://github.com/moonstar-x/discord-tts-bot) is a bot that uses the Google Translate API to utter the messages you send to the bot in any language. This bot has an image available on [Docker Hub](https://hub.docker.com/r/moonstarx/discord-tts-bot) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the bot's data will be saved.

```bash
mkdir ~/discord/discord-tts-bot
```

## Docker Compose

The bot will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  redis:
    image: redis:latest
    restart: unless-stopped
    ports:
     - 60000:6379
    volumes:
      - ./data:/data
    environment:
      - TZ=America/Guayaquil
    command: redis-server --save 60 1 --loglevel warning

  bot:
    image: moonstarx/discord-tts-bot:latest
    restart: unless-stopped
    depends_on:
      - redis
    environment:
      - DISCORD_TOKEN=<DISCORD_TOKEN_HERE>
      - DISCORD_OWNER_ID=<OWNER_ID_HERE>
      - DISCORD_DEFAULT_DISCONNECT_TIMEOUT=10
      - DISCORD_PROVIDER_TYPE=redis
      - DISCORD_REDIS_URL=redis://redis:6379
      - TZ=America/Guayaquil
```

## Running

Start up the bot with:

```bash
docker-compose up -d
```

That's it! The bot will auto-start on system startup and restart on failure.
