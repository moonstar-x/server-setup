# FreshRSS

[FreshRSS](https://www.freshrss.org/) is an RSS feed reader.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/freshrss](https://hub.docker.com/r/linuxserver/freshrss).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/freshrss
```

## Docker Compose

*FreshRSS* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  freshrss:
    image: ghcr.io/linuxserver/freshrss:latest
    restart: unless-stopped
    ports:
      - 11000:80
    volumes:
      - ./data:/config
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
