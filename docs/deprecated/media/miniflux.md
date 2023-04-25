# Miniflux

!!! warning "Deprecation Warning"
    This service was deprecated on the server in favor of [FreshRSS](../../services/media/freshrss.md) for its better web interface and third-party app support.

[Miniflux](https://miniflux.app/) is an RSS feed reader. This service has an official image available on [Docker Hub](https://hub.docker.com/r/miniflux/miniflux) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/miniflux
```

## Docker Compose

*Miniflux* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  miniflux:
    image: miniflux/miniflux:latest
    restart: unless-stopped
    ports:
      - 5190:8080
    environment:
      - TZ=America/Guayaquil
      - DATABASE_URL=postgres://miniflux:CHANGE_THIS@db/miniflux?sslmode=disable
      - RUN_MIGRATIONS=1
      - CREATE_ADMIN=1
      - ADMIN_USERNAME=CHANGE_THIS
      - ADMIN_PASSWORD=CHANGE_THIS

  db:
    image: postgres:13.7
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=miniflux
      - POSTGRES_PASSWORD=CHANGE_THIS
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 5190/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.