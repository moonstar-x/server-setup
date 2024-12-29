# Maybe

[Maybe](https://maybefinance.com) is a self-hosted finance tracker application.

There is an official image for this service that we'll use: [ghcr.io/maybe-finance/maybe](https://github.com/maybe-finance/maybe).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/maybe
```

## Docker Compose

*Maybe* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/maybe-finance/maybe:stable
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - maybe
    volumes:
      - ./storage:/rails/storage
    environment:
      TZ: America/Guayaquil
      SELF_HOSTED: true
      RAILS_FORCE_SSL: false
      RAILS_ASSUME_SSL: true
      GOOD_JOB_EXECUTION_MODE: async
      SECRET_KEY_BASE: SECRET_KEY
      DB_HOST: postgres
      POSTGRES_DB: maybe
      POSTGRES_USER: maybe
      POSTGRES_PASSWORD: DATABASE_PASSWORD
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.maybe.rule: Host(`subdomain.example.com`)
      traefik.http.routers.maybe.entrypoints: tunnel
      traefik.http.routers.maybe.service: maybe@docker
      traefik.http.services.maybe.loadbalancer.server.port: 3000
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.Host: '{host}'
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.X-Real-IP: '{clientip}'
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.X-Forwarded-For: '{clientip}'
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.X-Forwarded-Proto: '{scheme}'
      traefik.http.routers.maybe.middlewares: 'maybe-headers'

  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: maybe
      POSTGRES_USER: maybe
      POSTGRES_PASSWORD: DATABASE_PASSWORD

networks:
  proxy_external:
    external: true
```

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

!!! note
    Make sure to change `DATABASE_PASSWORD` to a custom secret value.

!!! note
    Make sure to change `SECRET_KEY` to a custom secret value.

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
