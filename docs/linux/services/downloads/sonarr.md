# Sonarr

[Sonarr](https://sonarr.tv/) is an RSS downloader focused on TV Shows.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/sonarr](https://hub.docker.com/r/linuxserver/sonarr).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/sonarr
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Sonarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  sonarr:
    image: ghcr.io/linuxserver/sonarr:latest
    restart: unless-stopped
    networks:
      - default
      - downloads_external
    ports:
      - 45300:8989
    volumes:
      - ./config:/config
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
      - /media/sata_2tb/Downloads:/downloads
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000

networks:
  downloads_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
