# OpenBooks

[OpenBooks](https://evan-buss.github.io/openbooks/) is an ebook downloader that can download from IRC Highway.

There is an official image for this service that we'll use: [ghcr.io/evan-buss/openbooks](https://github.com/evan-buss/openbooks/pkgs/container/openbooks).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/openbooks
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*OpenBooks* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/evan-buss/openbooks:latest
    restart: unless-stopped
    networks:
      default:
      downloads_external:
      proxy_external:
        aliases:
          - openbooks
    volumes:
      - /media/sata_2tb/Books/OpenBooks:/books
    command: --name USERNAME --persist
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.openbooks.rule: Host(`openbooks.home.example.com`, `openbooks.vpn.example.com`)
      traefik.http.routers.openbooks.entrypoints: local-https
      traefik.http.routers.openbooks.tls: true
      traefik.http.routers.openbooks.tls.certresolver: le
      traefik.http.routers.openbooks.service: openbooks@docker
      traefik.http.services.openbooks.loadbalancer.server.port: 80

networks:
  downloads_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    Make sure to replace `USERNAME` with something that can identify you in the IRC server.

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
