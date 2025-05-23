# Actual

[Actual](https://actualbudget.org/) is a self-hosted budgeting application.

There is an official image for this service that we'll use: [actualbudget/actual-server](https://hub.docker.com/r/actualbudget/actual-server/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/actual
```

## Docker Compose

*Actual* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: actualbudget/actual-server:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - actual
    volumes:
      - ./data:/data
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.actual.rule: Host(`${DOMAIN_ACTUAL}`)
      traefik.http.routers.actual.entrypoints: tunnel
      traefik.http.routers.actual.service: actual@docker
      traefik.http.services.actual.loadbalancer.server.port: 5006

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_ACTUAL=
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
