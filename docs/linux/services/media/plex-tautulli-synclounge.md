# Plex + Tautulli + Synclounge

!!! note
    This is a multi-service stack.

[Plex](https://plex.tv) is one of the most popular media server options out there. This media server has an official image available on [Docker Hub](https://hub.docker.com/r/plexinc/pms-docker/) which we'll use.

[Tautulli](https://tautulli.com/) is a monitoring tool for *Plex*. This service has an official image available on [Docker Hub](https://hub.docker.com/r/tautulli/tautulli) which we'll use.

[Synclounge](https://synclounge.tv/) is a tool that lets your Plex users watch something simultaneously. There is no official docker image for *Synclounge*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/synclounge).

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/plex
```

## Docker Compose

*Plex + Tautulli + Synclounge* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  plex:
    image: plexinc/pms-docker:latest
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./plex-config:/config
      - ./plex-transcode:/transcode
      - /media/usb_1tb:/media/usb_1tb
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
    environment:
      - TZ=America/Guayaquil
      - PLEX_UID=1000
      - PLEX_GID=1000

  tautulli:
    image: tautulli/tautulli:latest
    restart: unless-stopped
    depends_on:
      - plex
    volumes:
      - ./tautulli-config:/config
    ports:
      - 8181:8181
    environment:
      - TZ=America/Guayaquil
      - PUID=1000
      - PGID=1000

  synclounge:
    image: ghcr.io/linuxserver/synclounge:latest
    restart: unless-stopped
    depends_on:
      - plex
    ports:
      - 8088:8088
    environment:
      - TZ=America/Guayaquil
```

!!! note
    In the case of the `PLEX_UID` and `PLEX_GID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 32400/tcp
sudo ufw allow 8181/tcp
sudo ufw allow 8088/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
