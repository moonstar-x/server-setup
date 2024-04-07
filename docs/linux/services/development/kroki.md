# Kroki

[Kroki](https://kroki.io/) is a web service that exposes diagram services that allow you to generate dynamic diagrams from a `GET` or `POST` request. The diagrams are typically defined by text.

There is an official image for this service that we'll use: [yuzutech/kroki](https://hub.docker.com/r/yuzutech/kroki).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/kroki
```

## Docker Compose

*Kroki* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: yuzutech/kroki:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - kroki
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.kroki.rule: Host(`diagrams.home.arpa`)
      traefik.http.routers.kroki.entrypoints: local
      traefik.http.routers.kroki.service: kroki@docker
      traefik.http.services.kroki.loadbalancer.server.port: 8000

networks:
  proxy_external:
    external: true
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
