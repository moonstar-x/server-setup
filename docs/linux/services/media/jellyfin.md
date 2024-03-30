# Jellyfin

[Jellyfin](https://jellyfin.org/) is an open source version of the famous media server *Emby*.

There is an official image for this service that we'll use: [jellyfin/jellyfin](https://hub.docker.com/r/jellyfin/jellyfin).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/jellyfin
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `jellyfin_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create jellyfin_external
```

## Docker Compose

*Jellyfin* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    restart: unless-stopped
    user: 1000:1000
    networks:
      - default
      - jellyfin_external
    ports:
      - 60000:8096
    volumes:
      - ./jellyfin-config:/config
      - ./jellyfin-cache:/cache
      - /media/usb_4tb_2/Jellyfin:/media
    environment:
      TZ: America/Guayaquil

networks:
  jellyfin_external:
    external: true
```

!!! note
    In the case of the `user` directive, `1000:1000` corresponds to the user's `UID:GID`. You can find the values for your own user by running `id $whoami`.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
