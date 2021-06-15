# EmbyStat

!!! danger "Warning"
    This service requires [Jellyfin](jellyfin.md), you must set that up before continuing with this one.

[EmbyStat](https://github.com/mregni/EmbyStat) is an analytics service for Jellyfin/Emby. There is no official docker image for *EmbyStat*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/embystat).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/embystat
```

## Docker Compose

*EmbyStat* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  embystat:
    image: ghcr.io/linuxserver/embystat:latest
    restart: unless-stopped
    networks:
      - jellyfin
    volumes:
      - ./config:/config
    ports:
      - 6555:6555
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

networks:
  jellyfin:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 6555/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
