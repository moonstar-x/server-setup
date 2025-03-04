# Penpot

[Penpot](https://penpot.app) is a self-hosted UI/UX design tool similar to Figma.

There is an official image for this service that we'll use: [penpotapp/frontend](https://hub.docker.com/r/penpotapp/frontend), [penpotapp/backend](https://hub.docker.com/r/penpotapp/backend), and [penpotapp/exporter](https://hub.docker.com/r/penpotapp/exporter).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/penpot
```

## Docker Compose

*Penpot* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  penpot-frontend:
    image: penpotapp/frontend:latest
    restart: unless-stopped
    depends_on:
      - penpot-backend
      - penpot-exporter
    networks:
      default:
      proxy_external:
        aliases:
          - penpot
    volumes:
      - ./assets:/opt/data/assets
    environment:
      TZ: America/Guayaquil
      PENPOT_FLAGS: disable-registration enable-login-with-password enable-webhooks
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.penpot.rule: Host(`subdomain.example.com`)
      traefik.http.routers.penpot.entrypoints: public
      traefik.http.routers.penpot.service: penpot@docker
      traefik.http.services.penpot.loadbalancer.server.port: 8080

  penpot-backend:
    image: penpotapp/backend:latest
    restart: unless-stopped
    depends_on:
      - penpot-postgres
      - penpot-redis
    volumes:
      - ./assets:/opt/data/assets
    environment:
      TZ: America/Guayaquil
      PENPOT_FLAGS: disable-registration enable-login-with-password disable-email-verification disable-smtp enable-prepl-server enable-webhooks disable-telemetry
      PENPOT_SECRET_KEY: SUPER_SECRET_KEY
      PENPOT_PREPL_HOST: '0.0.0.0'
      PENPOT_PUBLIC_URI: https://subdomain.example.com
      PENPOT_DATABASE_URI: postgresql://penpot-postgres/penpot
      PENPOT_DATABASE_USERNAME: penpot
      PENPOT_DATABASE_PASSWORD: DATABASE_PASSWORD
      PENPOT_REDIS_URI: redis://penpot-redis/0
      PENPOT_ASSETS_STORAGE_BACKEND: assets-fs
      PENPOT_STORAGE_ASSETS_FS_DIRECTORY: /opt/data/assets
      PENPOT_TELEMETRY_ENABLED: false
    
  penpot-exporter:
    image: penpotapp/exporter:latest
    restart: unless-stopped
    environment:
      TZ: America/Guayaquil
      PENPOT_PUBLIC_URI: https://subdomain.example.com
      PENPOT_REDIS_URI: redis://penpot-redis/0

  penpot-postgres:
    image: postgres:15
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_INITDB_ARGS: --data-checksums
      POSTGRES_DB: penpot
      POSTGRES_USER: penpot
      POSTGRES_PASSWORD: DATABASE_PASSWORD

  penpot-redis:
    image: redis:7
    restart: unless-stopped
    environment:
      TZ: America/Guayaquil

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to change `DATABASE_PASSWORD` to a custom secret value.

!!! note
    Make sure to change `SUPER_SECRET_KEY` to a custom secret value.

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

!!! note
    The container names actually matter in this case. It seems someplace the names might be hardcoded and they really need to be named `penpot-backend` and `penpot-frontend`.

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Post-Installation

You may have noticed the `docker-compose.yml` file has the env variable `PENPOT_FLAGS` for the containers `penpot-frontend` and `penpot-backend`. By default we prefer to keep user registration disabled. This means you should probably remove the `disable-registration` flag the first time you start the service, create your users, and then re-add the flag to disable future registrations.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
