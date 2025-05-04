# Glitchtip

[Glitchtip](https://glitchtip.com) is a self-hosted error tracking similar to Sentry.

There is an official image for this service that we'll use: [glitchtip/glitchtip](https://hub.docker.com/r/glitchtip/glitchtip/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/glitchtip
```

## Docker Compose

*Glitchtip* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: glitchtip/glitchtip:latest
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    networks:
      default:
      proxy_external:
        aliases:
          - glitchtip
    volumes:
      - ./uploads:/code/uploads
    environment:
      TZ: America/Guayaquil
      DATABASE_URL: postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DATABASE}
      REDIS_URL: redis://cache:6379/1
      SECRET_KEY: ${SECRET_KEY}
      EMAIL_URL: consolemail://
      GLITCHTIP_DOMAIN: https://${DOMAIN_GLITCHTIP}
      DEFAULT_FROM_EMAIL: ${FROM_EMAIL}
      CELERY_WORKER_AUTOSCALE: 1,3
      ENABLE_USER_REGISTRATION: false
      ENABLE_ORGANIZATION_CREATION: false
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.glitchtip.rule: Host(`${DOMAIN_GLITCHTIP}`)
      traefik.http.routers.glitchtip.entrypoints: public
      traefik.http.routers.glitchtip.service: glitchtip@docker
      traefik.http.services.glitchtip.loadbalancer.server.port: 8080
      traefik.http.middlewares.glitchtip-headers.headers.customrequestheaders.X-Forwarded-Proto: https
      traefik.http.middlewares.glitchtip-headers.headers.customrequestheaders.Host: ${DOMAIN_GLITCHTIP}
      traefik.http.routers.glitchtip.middlewares: glitchtip-headers

  worker:
    image: glitchtip/glitchtip:latest
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    command: ./bin/run-celery-with-beat.sh
    volumes:
      - ./uploads:/code/uploads
    environment:
      TZ: America/Guayaquil
      DATABASE_URL: postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DATABASE}
      REDIS_URL: redis://cache:6379/1
      SECRET_KEY: ${SECRET_KEY}
      EMAIL_URL: consolemail://
      GLITCHTIP_DOMAIN: https://${DOMAIN_GLITCHTIP}
      DEFAULT_FROM_EMAIL: ${FROM_EMAIL}
      CELERY_WORKER_AUTOSCALE: 1,3
      ENABLE_USER_REGISTRATION: false
      ENABLE_ORGANIZATION_CREATION: false

  migrate:
    image: glitchtip/glitchtip:latest
    restart: no
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    command: ./bin/run-migrate.sh
    environment:
      TZ: America/Guayaquil
      DATABASE_URL: postgres://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DATABASE}
      REDIS_URL: redis://cache:6379/1
      SECRET_KEY: ${SECRET_KEY}
      EMAIL_URL: consolemail://
      GLITCHTIP_DOMAIN: https://${DOMAIN_GLITCHTIP}
      DEFAULT_FROM_EMAIL: ${FROM_EMAIL}
      CELERY_WORKER_AUTOSCALE: 1,3
      ENABLE_USER_REGISTRATION: false
      ENABLE_ORGANIZATION_CREATION: false

  db:
    image: postgres:17
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: ${POSTGRES_DATABASE}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

  cache:
    image: valkey/valkey:latest
    restart: unless-stopped
    environment:
      TZ: America/Guayaquil

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
POSTGRES_DATABASE=
POSTGRES_USER=
POSTGRES_PASSWORD=

SECRET_KEY=
FROM_EMAIL=

DOMAIN_GLITCHTIP=
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
