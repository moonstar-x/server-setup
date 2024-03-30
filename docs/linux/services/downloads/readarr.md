# Readarr

[Readarr](https://readarr.com/) is an RSS downloader focused on ebooks and audiobooks.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/readarr](https://hub.docker.com/r/linuxserver/readarr).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/readarr
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Readarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  readarr:
    image: ghcr.io/linuxserver/readarr:develop
    restart: unless-stopped
    networks:
      - default
      - downloads_external
    ports:
      - 45700:8787
    volumes:
      - ./config:/config
      - /media/sata_2tb/Books:/books
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
