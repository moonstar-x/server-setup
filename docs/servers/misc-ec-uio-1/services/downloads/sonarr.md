# Sonarr

[Sonarr](https://sonarr.tv/) is an RSS downloader focused on TV Shows.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/sonarr](https://hub.docker.com/r/linuxserver/sonarr).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/sonarr
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Sonarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/sonarr:latest
    restart: unless-stopped
    networks:
      default:
      downloads_external:
      proxy_external:
        aliases:
          - sonarr
    volumes:
      - ./config:/config
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
      - /media/sata_2tb/Downloads:/downloads
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.sonarr.rule: Host(`${DOMAIN_SONARR_LOCAL}`) || Host(`${DOMAIN_SONARR_VPN}`)
      traefik.http.routers.sonarr.entrypoints: local-https
      traefik.http.routers.sonarr.tls: true
      traefik.http.routers.sonarr.tls.certresolver: le
      traefik.http.routers.sonarr.service: sonarr@docker
      traefik.http.services.sonarr.loadbalancer.server.port: 8989

networks:
  downloads_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_SONARR_LOCAL=
DOMAIN_SONARR_VPN=
```

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
