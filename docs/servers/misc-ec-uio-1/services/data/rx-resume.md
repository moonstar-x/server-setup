# Reactive Resume

[Reactive Resume](https://rxresu.me/) is a tool to easily create beautiful CVs.

There is an official image for this service that we'll use: [amruthpillai/reactive-resume](https://hub.docker.com/r/amruthpillai/reactive-resume).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/rxresume
```

## Docker Compose

*Reactive Resume* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  db:
    image: postgres:15-alpine
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: ${POSTGRES_DATABASE}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

  api:
    image: amruthpillai/reactive-resume:server-latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - rxresume_api
    depends_on:
      - db
    environment:
      TZ: America/Guayaquil
      PUBLIC_URL: https://${DOMAIN_RXRESUME_WEB}
      PUBLIC_SERVER_URL: https://${DOMAIN_RXRESUME_API}
      POSTGRES_DB: ${POSTGRES_DATABASE}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      SECRET_KEY: ${SECRET_KEY}
      POSTGRES_HOST: db
      POSTGRES_PORT: 5432
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRY_TIME: 604800
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.rxresume_api.rule: Host(`${DOMAIN_RXRESUME_API}`)
      traefik.http.routers.rxresume_api.entrypoints: local-https
      traefik.http.routers.rxresume_api.tls: true
      traefik.http.routers.rxresume_api.tls.certresolver: le
      traefik.http.routers.rxresume_api.service: rxresume_api@docker
      traefik.http.services.rxresume_api.loadbalancer.server.port: 3100

  web:
    image: amruthpillai/reactive-resume:client-latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - rxresume_web
    depends_on:
      - api
    environment:
      TZ: America/Guayaquil
      PUBLIC_URL: https://${DOMAIN_RXRESUME_WEB}
      PUBLIC_SERVER_URL: https://${DOMAIN_RXRESUME_API}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.rxresume_web.rule: Host(`${DOMAIN_RXRESUME_WEB}`)
      traefik.http.routers.rxresume_web.entrypoints: local-https
      traefik.http.routers.rxresume_web.tls: true
      traefik.http.routers.rxresume_web.tls.certresolver: le
      traefik.http.routers.rxresume_web.service: rxresume_web@docker
      traefik.http.services.rxresume_web.loadbalancer.server.port: 3000

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
JWT_SECRET=

DOMAIN_RXRESUME_WEB=
DOMAIN_RXRESUME_API=
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
