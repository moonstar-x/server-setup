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
  bot:
    image: moonstarx/discord-tts-bot:latest
    restart: unless-stopped
    network_mode: host
    environment:
      - DISCORD_TOKEN=<DISCORD_TOKEN_HERE>
      - PREFIX=$$
      - TZ=America/Guayaquil
```

!!! note
    The prefix in this case is `$`, *Docker Compose* requires to escape any `$` symbol with another `$`.

## Running

Start up the bot with:

```bash
docker-compose up -d
```

That's it! The bot will auto-start on system startup and restart on failure.
