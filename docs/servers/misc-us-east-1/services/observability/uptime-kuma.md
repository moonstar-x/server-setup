# Uptime Kuma

[Uptime Kuma](https://uptime.kuma.pet/) is a status monitoring service that can expose a page with your services' uptime status.

There is an official image for this service that we'll use: [louislam/uptime-kuma](https://hub.docker.com/r/louislam/uptime-kuma).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/observability/uptime-kuma
```

## Docker Compose

*Uptime Kuma* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: louislam/uptime-kuma:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - uptime-kuma
    volumes:
      - ./data:/app/data
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.uptime-kuma.rule: Host(`${DOMAIN_UPTIME_KUMA}`)
      traefik.http.routers.uptime-kuma.entrypoints: tunnel
      traefik.http.routers.uptime-kuma.service: uptime-kuma@docker
      traefik.http.services.uptime-kuma.loadbalancer.server.port: 3001

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_UPTIME_KUMA=
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
