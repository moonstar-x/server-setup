# Nextcloud

[Nextcloud](https://nextcloud.com/) is a self-hosted cloud data server, useful for keeping documents in your own server.

There is an official image for this service that we'll use: [nextcloud](https://hub.docker.com/r/_/nextcloud).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/nextcloud
```

## Dockerfile

Despite the service having an official image available, it lacks a dependency necessary to allow for SMB shares as external storage.

The following is the (tiny) `Dockerfile` that will be used for this service:

```docker
FROM nextcloud:stable

RUN apt-get update && apt-get install -y procps smbclient && rm -rf /var/lib/apt/lists/*
```

## Docker Compose

*Nextcloud* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    build: .
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - nextcloud
    depends_on:
      - db
    volumes:
      - /media/sata_2tb/Nextcloud:/var/www/html
    environment:
      TZ: America/Guayaquil
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_HOST: db
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.nextcloud.rule: Host(`${DOMAIN_NEXTCLOUD}`)
      traefik.http.routers.nextcloud.entrypoints: public
      traefik.http.routers.nextcloud.service: nextcloud@docker
      traefik.http.services.nextcloud.loadbalancer.server.port: 80
      traefik.http.middlewares.limit.buffering.memResponseBodyBytes: 2000000000

  db:
    image: mariadb:10.5
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/mysql
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    environment:
      TZ: America/Guayaquil
      MYSQL_ROOT_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}

  cron:
    image: nextcloud:stable
    restart: unless-stopped
    depends_on:
      - web
    volumes:
      - /media/sata_2tb/Nextcloud:/var/www/html
    entrypoint: /cron.sh
    environment:
      TZ: America/Guayaquil

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
MYSQL_DATABASE=
MYSQL_USER=
MYSQL_PASSWORD=

DOMAIN_NEXTCLOUD=
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
