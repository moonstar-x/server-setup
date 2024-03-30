# Kavita

[Kavita](https://www.kavitareader.com/) is an eBook server for PDFs and EPUBs.

There is an official image for this service that we'll use: [jvmilazz0/kavita](https://hub.docker.com/r/jvmilazz0/kavita).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/kavita
```

## Docker Compose

*Kavita* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  kavita:
    image: jvmilazz0/kavita:latest
    restart: unless-stopped
    ports:
      - 12000:5000
    volumes:
      - ./data:/kavita/config
      - /media/sata_2tb/Books:/books
    environment:
      TZ: America/Guayaquil
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
