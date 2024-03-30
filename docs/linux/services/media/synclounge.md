# Synclounge

[Synclounge](https://synclounge.tv/) is a tool that lets your Plex users watch something simultaneously.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/synclounge](https://hub.docker.com/r/linuxserver/synclounge).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/synclounge
```

## Docker Compose

*Synclounge* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/synclounge:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - synclounge
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.synclounge.rule: Host(`subdomain.example.com`)
      traefik.http.routers.synclounge.entrypoints: public
      traefik.http.routers.synclounge.service: synclounge@docker
      traefik.http.services.synclounge.loadbalancer.server.port: 8088

networks:
  proxy_external:
    external: true
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
