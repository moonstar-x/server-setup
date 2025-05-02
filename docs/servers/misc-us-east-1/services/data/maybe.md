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
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./storage:/rails/storage
    environment:
      TZ: America/Guayaquil
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DATABASE}
      SECRET_KEY_BASE: ${SECRET_KEY_BASE}
      SELF_HOSTED: true
      RAILS_FORCE_SSL: false
      RAILS_ASSUME_SSL: true
      DB_HOST: postgres
      DB_PORT: 5432
      REDIS_URL: redis://redis:6379/1
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.maybe.rule: Host(`${DOMAIN_MAYBE}`)
      traefik.http.routers.maybe.entrypoints: tunnel
      traefik.http.routers.maybe.service: maybe@docker
      traefik.http.services.maybe.loadbalancer.server.port: 3000
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.Host: '{host}'
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.X-Real-IP: '{clientip}'
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.X-Forwarded-For: '{clientip}'
      traefik.http.middlewares.maybe-headers.headers.customRequestHeaders.X-Forwarded-Proto: '{scheme}'
      traefik.http.routers.maybe.middlewares: 'maybe-headers'

  worker:
    image: ghcr.io/maybe-finance/maybe:latest
    restart: unless-stopped
    command: bundle exec sidekiq
    depends_on:
      redis:
        condition: service_healthy
    environment:
      TZ: America/Guayaquil
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DATABASE}
      SECRET_KEY_BASE: ${SECRET_KEY_BASE}
      SELF_HOSTED: true
      RAILS_FORCE_SSL: false
      RAILS_ASSUME_SSL: true
      DB_HOST: postgres
      DB_PORT: 5432
      REDIS_URL: redis://redis:6379/1

  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB" ]
      interval: 5s
      timeout: 5s
      retries: 5
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: ${POSTGRES_DATABASE}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

  redis:
    image: redis:7.4-rc2-alpine
    restart: unless-stopped
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
      interval: 5s
      timeout: 5s
      retries: 5
    volumes:
      - ./redis:/data

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
SECRET_KEY_BASE=
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DATABASE=
DOMAIN_MAYBE=
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
