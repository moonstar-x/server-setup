# Ombi

!!! danger "Warning"
    This service requires [Transmission](transmission.md), you must set that up before continuing with this one.

[Ombi](https://ombi.io/) is a media request tracker, useful for when you share a Plex server or similar with family and friends. There is no official docker image for *Ombi*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/ombi).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/ombi
```

## Docker Compose

*Ombi* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  ombi:
    image: ghcr.io/linuxserver/ombi:latest
    restart: unless-stopped
    networks:
      - downloader
    volumes:
      - ./config:/config
    ports:
      - 3579:3579
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

networks:
  downloader:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 3579/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
