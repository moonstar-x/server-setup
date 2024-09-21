# Bazarr

[Bazarr](https://www.bazarr.media/) is an RSS downloader focused on subtitles.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/bazarr](https://hub.docker.com/r/linuxserver/bazarr).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/bazarr
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Bazarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/bazarr:latest
    restart: unless-stopped
    networks:
      default:
      downloads_external:
      proxy_external:
        aliases:
          - bazarr
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
      traefik.http.routers.bazarr.rule: Host(`bazarr.alpha.example.com`) || Host(`bazarr.alpha.home.example.com`)
      traefik.http.routers.bazarr.entrypoints: local-https
      traefik.http.routers.bazarr.tls: true
      traefik.http.routers.bazarr.tls.certresolver: le
      traefik.http.routers.bazarr.service: bazarr@docker
      traefik.http.services.bazarr.loadbalancer.server.port: 6767

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
