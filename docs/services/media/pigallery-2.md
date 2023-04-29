# PiGallery 2

[PiGallery 2](https://bpatrik.github.io/pigallery2/) is a simple image gallery. This service has an official image available on [Docker Hub](https://hub.docker.com/r/bpatrik/pigallery2) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/pigallery2
```

## Docker Compose

*PiGallery 2* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  pigallery2:
    image: bpatrik/pigallery2:latest
    restart: unless-stopped
    volumes:
      - ./config:/app/data/config
      - ./data:/app/data/db
      - ./cache:/app/data/tmp
      - /media/usb_4tb_2/Images:/app/data/images:ro
    ports:
      - 20500:80
    environment:
      - TZ=America/Guayaquil
      - NODE_ENV=productio
```

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 20500/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
