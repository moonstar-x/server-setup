# Pairdrop

[Pairdrop](https://pairdrop.net/) is a peer-to-peer file sharing service that mimics Apple's AirDrop functionality through a web interface.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/pairdrop](https://hub.docker.com/r/linuxserver/pairdrop).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/pairdrop
```

## Docker Compose

*Pairdrop* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/pairdrop:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - pairdrop
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.pairdrop.rule: Host(`pairdrop.alpha.example.com`) || Host(`pairdrop.alpha.home.example.com`)
      traefik.http.routers.pairdrop.entrypoints: local-https
      traefik.http.routers.pairdrop.tls: true
      traefik.http.routers.pairdrop.tls.certresolver: le
      traefik.http.routers.pairdrop.service: pairdrop@docker
      traefik.http.services.pairdrop.loadbalancer.server.port: 3000

networks:
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
