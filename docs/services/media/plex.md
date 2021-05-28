# Plex

[Plex](https://plex.tv) one of the most popular media server options out there. This media server has an official image available on [Docker Hub](https://hub.docker.com/r/plexinc/pms-docker/) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/plex
```

## Docker Compose

*Plex* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  plex:
    image: plexinc/pms-docker:latest
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./config:/config
      - ./transcode:/transcode
      - /media/usb_1tb:/media/usb_1tb
      - /media/usb_4tb:/media/usb_4tb
    environment:
      - TZ=America/Guayaquil
      - PLEX_UID=1000
      - PLEX_GID=1000
```

!!! note
    In the case of the `PLEX_UID` and `PLEX_GID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 32400/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
