# Umami

[Umami](https://umami.is/) is a web analytics service.

There is an official image for this service that we'll use: [ghcr.io/umami-software](https://github.com/umami-software/umami).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/analytics/umami
```

## Docker Compose

*Umami* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/umami-software/umami:postgresql-latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - umami
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD-SHELL", "curl http://localhost:3000/api/heartbeat"]
      interval: 5s
      timeout: 5s
      retries: 5
    environment:
      TZ: America/Guayaquil
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DATABASE}
      DATABASE_TYPE: postgresql
      APP_SECRET: ${APP_SECRET}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.umami-http.rule: Host(`${DOMAIN_UMAMI}`)
      traefik.http.routers.umami-http.entrypoints: http
      traefik.http.routers.umami-http.middlewares: umami-redirectscheme
      traefik.http.routers.umami-http.service: umami@docker
      traefik.http.routers.umami-https.rule: Host(`${DOMAIN_UMAMI}`)
      traefik.http.routers.umami-https.entrypoints: https
      traefik.http.routers.umami-https.service: umami@docker
      traefik.http.routers.umami-https.middlewares: umami-headers
      traefik.http.routers.umami-https.tls: true
      traefik.http.routers.umami-https.tls.certresolver: le
      traefik.http.services.umami.loadbalancer.server.port: 3000
      traefik.http.middlewares.umami-redirectscheme.redirectscheme.scheme: https
      traefik.http.middlewares.umami-redirectscheme.redirectscheme.permanent: true
      traefik.http.middlewares.umami-headers.headers.customrequestheaders.X-Forwarded-Proto: https
      traefik.http.middlewares.umami-headers.headers.customrequestheaders.Host: ${DOMAIN_UMAMI}

  db:
    image: postgres:15-alpine
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DATABASE}"]
      interval: 5s
      timeout: 5s
      retries: 5
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: ${POSTGRES_DATABASE}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
APP_SECRET=
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DATABASE=
DOMAIN_UMAMI=
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

## Post Configuration

The default credentials:

* User: `admin`
* Password: `umami`

Make sure to change these credentials or remove them altogether.
