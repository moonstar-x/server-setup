# Prowlarr

[Prowlarr](https://prowlarr.com/) is an indexer for -arr software.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/prowlarr](https://hub.docker.com/r/linuxserver/prowlarr).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/prowlarr
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Prowlarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/prowlarr:latest
    restart: unless-stopped
    networks:
      default:
      downloads_external:
      proxy_external:
        aliases:
          - prowlarr
    volumes:
      - ./config:/config
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.prowlarr.rule: Host(`prowlarr.alpha.example.com`) || Host(`prowlarr.alpha.home.example.com`)
      traefik.http.routers.prowlarr.entrypoints: local-https
      traefik.http.routers.prowlarr.tls: true
      traefik.http.routers.prowlarr.tls.certresolver: le
      traefik.http.routers.prowlarr.service: prowlarr@docker
      traefik.http.services.prowlarr.loadbalancer.server.port: 9696

networks:
  downloads_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
