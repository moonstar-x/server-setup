# Jellyfin

[Jellyfin](https://jellyfin.org/) is an open source of the famous media server *Emby*. This media server has an official image available on [Docker Hub](https://hub.docker.com/r/jellyfin/jellyfin) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/jellyfin
```

We'll also need to create a custom network to allow other containers to communicate with this one.

```bash
docker network create jellyfin
```

## Docker Compose

*Jellyfin* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    user: 1000:1000
    restart: unless-stopped
    networks:
      - jellyfin
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /media/sata_2tb/Jellyfin:/media
    ports:
      - 8096:8096
    environment:
      - TZ=America/Guayaquil

networks:
  jellyfin:
    external: true
```

!!! note
    In the case of the `user` directive, `1000:1000` corresponds to the user's `UID:GID`. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8086/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
