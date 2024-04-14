# Jellystat

[Jellystat](https://github.com/CyferShepard/Jellystat) is an analytics service for Jellyfin.

There is an official image for this service that we'll use: [cyfershepard/jellystat](https://hub.docker.com/r/cyfershepard/jellystat).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/jellystat
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `jellyfin_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create jellyfin_external
```

## Docker Compose

*Jellystat* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: cyfershepard/jellystat:latest
    restart: unless-stopped
    depends_on:
      - db
    networks:
      default:
      jellyfin_external:
      proxy_external:
        aliases:
          - jellystat
    volumes:
      - ./backup-data:/app/backend/backup-data
    environment:
      TZ: America/Guayaquil
      POSTGRES_USER: DATABASE_USER
      POSTGRES_PASSWORD: DATABASE_PASSWORD
      POSTGRES_IP: db
      POSTGRES_PORT: 5432
      JWT_SECRET: JWT_SECRET
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.jellystat.rule: Host(`jellystat.home.example.com`, `jellystat.vpn.example.com`)
      traefik.http.routers.jellystat.entrypoints: local-https
      traefik.http.routers.jellystat.tls: true
      traefik.http.routers.jellystat.tls.certresolver: le
      traefik.http.routers.jellystat.service: jellystat@docker
      traefik.http.services.jellystat.loadbalancer.server.port: 3000

  db:
    image: postgres:15.2
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: jellystat
      POSTGRES_USER: DATABASE_USER
      POSTGRES_PASSWORD: DATABASE_PASSWORD

networks:
  jellyfin_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    Make sure to change `DATABASE_USER` and `DATABASE_PASSWORD` to a custom secret value.

!!! note
    Make sure to change `JWT_SECRET` to a custom secret value. You can generate this with `openssl rand -hex 32`.

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
