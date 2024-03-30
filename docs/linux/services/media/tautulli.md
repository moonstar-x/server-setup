# Tautulli

[Tautulli](https://tautulli.com/) is a monitoring tool for *Plex*.

There is an official image for this service that we'll use [tautulli/tautulli](https://hub.docker.com/r/tautulli/tautulli).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/tautulli
```

## Docker Compose

*Tautulli* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  tautulli:
    image: tautulli/tautulli:latest
    restart: unless-stopped
    ports:
      - 10000:8181
    volumes:
      - ./config:/config
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
