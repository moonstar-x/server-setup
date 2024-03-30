# PiGallery 2

[PiGallery 2](https://bpatrik.github.io/pigallery2/) is a simple image gallery.

There is an official image for this service that we'll use: [bpatrik/pigallery2](https://hub.docker.com/r/bpatrik/pigallery2).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/pigallery2
```

## Docker Compose

*PiGallery 2* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  pigallery2:
    image: bpatrik/pigallery2:1.9.5
    restart: unless-stopped
    ports:
      - 61000:80
    volumes:
      - ./config:/app/data/config
      - ./data:/app/data/db
      - /media/usb_4tb_2/Gallery/Caches:/app/data/tmp
      - /media/usb_4tb_2/Gallery/Images:/app/data/images:ro
    environment:
      TZ: America/Guayaquil
      NODE_ENV: production
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
