# Jackett

!!! danger "Warning"
    This service requires [Transmission](transmission.md), you must set that up before continuing with this one.

[Jackett](https://github.com/Jackett/Jackett) is a torrent indexer that standardizes the shape of the data from multiple indexers to make it possible to use originally unsupported indexers on services like *Sonarr*. There is no official docker image for *Jackett*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/jackett/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/jackett
```

## Docker Compose

*Jackett* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  jackett:
    image: ghcr.io/linuxserver/jackett:latest
    restart: unless-stopped
    networks:
      - downloader
    volumes:
      - ./config:/config
    ports:
      - 9117:9117
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil
      - AUTO_UPDATE=true

networks:
  downloader:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 9117/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
