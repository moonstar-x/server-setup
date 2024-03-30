# EmbyStat

[EmbyStat](https://github.com/mregni/EmbyStat) is an analytics service for Jellyfin/Emby.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/embystat](https://hub.docker.com/r/linuxserver/embystat).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/embystat
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `jellyfin_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create jellyfin_external
```

## Docker Compose

*EmbyStat* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/embystat:latest
    restart: unless-stopped
    networks:
      default:
      jellyfin_external:
      proxy_external:
        aliases:
          - embystat
    volumes:
      - ./config:/config
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.embystat.rule: Host(`embystat.home.arpa`)
      traefik.http.routers.embystat.entrypoints: local
      traefik.http.routers.embystat.service: embystat@docker
      traefik.http.services.embystat.loadbalancer.server.port: 6555

networks:
  jellyfin_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
