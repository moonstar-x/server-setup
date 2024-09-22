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
      DATABASE_URL: postgresql://DATABASE_USER:DATABASE_PASSWORD@db:5432/umami
      DATABASE_TYPE: postgresql
      APP_SECRET: APP_SECRET
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.umami.rule: Host(`subdomain.example.com`)
      traefik.http.routers.umami.entrypoints: tunnel
      traefik.http.routers.umami.service: umami@docker
      traefik.http.services.umami.loadbalancer.server.port: 3000

  db:
    image: postgres:15-alpine
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: umami
      POSTGRES_USER: DATABASE_USER
      POSTGRES_PASSWORD: DATABASE_PASSWORD

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to change `DATABASE_USER` and `DATABASE_PASSWORD` to a custom secret value.

!!! note
    Make sure to change `APP_SECRET` to a custom secret value.

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

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
