# Romm

[Romm](https://github.com/rommapp/romm) is a frontend to manage and even play emulator roms.

There is an official image for this service that we'll use: [rommapp/romm](https://hub.docker.com/r/rommapp/romm).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/games/romm
```

## Docker Compose

*Romm* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: rommapp/romm:latest
    restart: unless-stopped
    depends_on:
      - db
    networks:
      default:
      proxy_external:
        aliases:
          - romm
    volumes:
      - ./resources:/romm/resources
      - ./redis:/romm/redis-data
      - /media/sata_2tb/Retroid:/romm/library
      - ./assets:/romm/assets
      - ./config:/romm/config
    environment:
      TZ: America/Guayaquil
      DB_HOST: db
      DB_NAME: romm
      DB_USER: DATABASE_USER
      DB_PASSWD: DATABASE_PASSWORD
      IGDB_CLIENT_ID: IGDB_CLIENT_ID
      IGDB_CLIENT_SECRET: IGDB_CLIENT_SECRET
      ROMM_AUTH_SECRET_KEY: AUTH_SECRET_KEY
      ROMM_AUTH_USERNAME: AUTH_USERNAME
      ROMM_AUTH_PASSWORD: AUTH_PASSWORD
      DISABLE_CSRF_PROTECTION: true
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.romm.rule: Host(`roms.alpha.example.com`) || Host(`roms.alpha.home.example.com`)
      traefik.http.routers.romm.entrypoints: local-https
      traefik.http.routers.romm.tls: true
      traefik.http.routers.romm.tls.certresolver: le
      traefik.http.routers.romm.service: romm@docker
      traefik.http.services.romm.loadbalancer.server.port: 8080

  db:
    image: mariadb:10
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/mysql
    environment:
      TZ: America/Guayaquil
      MYSQL_ROOT_PASSWORD: DATABASE_PASSWORD
      MYSQL_DATABASE: romm
      MYSQL_USER: DATABASE_USER
      MYSQL_PASSWORD: DATABASE_PASSWORD

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to change `DATABASE_USER`, `DATABASE_PASSWORD` to a custom secret value.

!!! note
    Make sure to change `AUTH_SECRET_KEY` to a custom secret value. You can generate this with `openssl rand -hex 32`. Also replace `AUTH_USERNAME` and `AUTH_PASSWORD` to something secret.

!!! note
    Make sure to change `IGDB_CLIENT_ID` and `IGDB_CLIENT_SECRET` with a client ID and secret generated from [twitch.tv](https://dev.twitch.tv/console/apps).

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
