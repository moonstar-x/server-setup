# Gitea

[Gitea](https://gitea.io/en-us/) is a self-hosted git server, useful for having a private VCS solution.

There is an official image for this service that we'll use: [gitea/gitea](https://hub.docker.com/r/gitea/gitea/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/observability/gitea
```

### Proxy Set-up

Since this service will be exposed to the Internet through [Traefik](../networking/traefik.md), we'll need to add this service to the `proxy_external` network. Make sure you have it set up before configuring this service.

## Docker Compose

*Gitea* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  gitea:
    image: gitea/gitea:latest
    restart: unless-stopped
    depends_on:
      - db
    networks:
      - default
      - proxy_external
    volumes:
      - ./data:/data
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.gitea.rule: Host(`subdomain.example.com`)
      traefik.http.routers.gitea.entrypoints: public
      traefik.http.routers.gitea.service: gitea@docker
      traefik.http.services.gitea.loadbalancer.server.port: 3000

  db:
    image: mariadb:10
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/mysql
    environment:
      TZ: America/Guayaquil
      MYSQL_ROOT_PASSWORD: DATABASE_PASSWORD
      MYSQL_DATABASE: gitea
      MYSQL_USER: gitea
      MYSQL_PASSOWRD: DATABASE_PASSWORD

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to change `DATABASE_PASSWORD` to a custom secret value.

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

## Post-Installation

nce you have started the server once, edit the config file located inside the `data` volume:

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
