# Wakapi

[Wakapi](https://wakapi.dev) is a self-hosted code statistics tracker similar to [Wakatime](https://wakatime.com).

There is an official image for this service that we'll use: [ghcr.io/muety/wakapi](https://github.com/muety/wakapi).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/wakapi
```

## Docker Compose

*Wakapi* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/muety/wakapi:latest
    restart: unless-stopped
    depends_on:
      - db
    networks:
      default:
      proxy_external:
        aliases:
          - wakapi
    environment:
      TZ: America/Guayaquil
      ENVIRONMENT: prod
      WAKAPI_LEADERBOARD_ENABLED: false
      WAKAPI_PUBLIC_URL: https://${DOMAIN_WAKAPI}
      WAKAPI_PASSWORD_SALT: ${SALT}
      WAKAPI_ALLOW_SIGNUP: false
      WAKAPI_DISABLE_FRONTPAGE: true
      WAKAPI_EXPOSE_METRICS: true
      WAKAPI_DB_TYPE: postgres
      WAKAPI_DB_NAME: ${POSTGRES_DB}
      WAKAPI_DB_USER: ${POSTGRES_USER}
      WAKAPI_DB_PASSWORD: ${POSTGRES_PASSWORD}
      WAKAPI_DB_HOST: db
      WAKAPI_DB_PORT: 5432
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.wakapi.rule: Host(`${DOMAIN_WAKAPI}`)
      traefik.http.routers.wakapi.entrypoints: public
      traefik.http.routers.wakapi.service: wakapi@docker
      traefik.http.services.wakapi.loadbalancer.server.port: 3000

  db:
    image: postgres:17
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
SALT=
POSTGRES_DB=
POSTGRES_USER=
POSTGRES_PASSWORD=
DOMAIN_WAKAPI=
```

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Post-Installation

You'll notice that the `WAKAPI_ALLOW_SIGNUP` variable is set to `false` in the `docker-compose.yml` file. Consider setting this to `true` the first time you start the service up, sign up with your user, and then re-disable this.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
