# Gitea

[Gitea](https://gitea.io/en-us/) is a self-hosted git server, useful for having a private VCS solution.

There is an official image for this service that we'll use: [gitea/gitea](https://hub.docker.com/r/gitea/gitea/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/gitea
```

## Docker Compose

*Gitea* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: gitea/gitea:latest
    restart: unless-stopped
    depends_on:
      - db
    networks:
      default:
      proxy_external:
        aliases:
          - gitea
    volumes:
      - ./data:/data
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.gitea.rule: Host(`${DOMAIN_GITEA}`)
      traefik.http.routers.gitea.entrypoints: public
      traefik.http.routers.gitea.service: gitea@docker
      traefik.http.services.gitea.loadbalancer.server.port: 3000
      traefik.http.middlewares.gitea-headers.headers.customrequestheaders.X-Forwarded-Proto: https
      traefik.http.middlewares.gitea-headers.headers.customrequestheaders.Host: ${DOMAIN_GITEA}
      traefik.http.routers.gitea.middlewares: gitea-headers

  db:
    image: postgres:14
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/postgresql/data
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
POSTGRES_DB=
POSTGRES_USER=
POSTGRES_PASSWORD=

DOMAIN_GITEA=
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

Once you have started the server once, edit the config file located inside the `data` volume:

```bash
nano data/gitea/conf/app.ini
```

And make sure to have the following lines:

```text
[service]
DISABLE_REGISTRATION = true
```

This will make sure that nobody else can register into your server without your knowledge.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
