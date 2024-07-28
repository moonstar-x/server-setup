# RSSHub

[RSSHub](https://docs.rsshub.app/) is a RSS bridge that generates RSS feeds for services that normally do not support them.

There is an official image for this service that we'll use: [diygod/rsshub](https://hub.docker.com/r/diygod/rsshub).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/automation/rsshub
```

## Docker Compose

*RSSHub* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: diygod/rsshub:latest
    restart: unless-stopped
    depends_on:
      - redis
      - browserless
    networks:
      default:
      proxy_external:
        aliases:
          - rsshub
    environment:
      TZ: America/Guayaquil
      NODE_ENV: production
      CACHE_TYPE: redis
      REDIS_URL: redis://redis:6379
      PUPPETEER_WS_ENDPOINT: ws://browserless:3000
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.rsshub.rule: Host(`rss.home.example.com`) || Host(`rss.vpn.example.com`)
      traefik.http.routers.rsshub.entrypoints: local-https
      traefik.http.routers.rsshub.tls: true
      traefik.http.routers.rsshub.tls.certresolver: le
      traefik.http.routers.rsshub.service: rsshub@docker
      traefik.http.services.rsshub.loadbalancer.server.port: 1200

  browserless:
    image: browserless/chrome:latest
    restart: unless-stopped
    ulimits:
      core:
        hard: 0
        soft: 0

  redis:
    image: redis:alpine
    restart: unless-stopped
    volumes:
      - ./redis:/data

networks:
  proxy_external:
    external: true
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
