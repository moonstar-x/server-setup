# Sonarr

!!! danger "Warning"
    This service requires [Transmission](transmission.md), you must set that up before continuing with this one.

[Sonarr](https://sonarr.tv/) is an RSS downloader focused on TV Shows. There is no official docker image for *Sonarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/sonarr).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/sonarr
```

## Docker Compose

*Sonarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  sonarr:
    image: ghcr.io/linuxserver/sonarr:latest
    restart: unless-stopped
    networks:
      - downloader
    volumes:
      - ./config:/config
      - /media/usb_1tb:/media/usb_1tb
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
      - /media/sata_2tb/Downloads:/downloads
    ports:
      - 8989:8989
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
sudo ufw allow 8989/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
