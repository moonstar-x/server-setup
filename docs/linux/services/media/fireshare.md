# Fireshare

[Fireshare](https://github.com/ShaneIsrael/fireshare) is a web service that allows you to share game clips.

There is an official image for this service that we'll use: [shaneisrael/fireshare](https://hub.docker.com/r/shaneisrael/fireshare/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/fireshare
```

## Docker Compose

*Fireshare* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: shaneisrael/fireshare:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - fireshare
    volumes:
      - ./data:/data
      - ./processed:/processed
      - ./videos:/videos
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
      ADMIN_USERNAME: AUTH_USERNAME
      ADMIN_PASSWORD: AUTH_PASSWORD
      SECRET_KEY: AUTH_SECRET
      MINUTES_BETWEEN_VIDEO_SCANS: 5
      THUMBNAIL_VIDEO_LOCATION: 0
      DOMAIN: subdomain.example.com
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.fireshare.rule: Host(`subdomain.example.com`)
      traefik.http.routers.fireshare.entrypoints: public
      traefik.http.routers.fireshare.service: fireshare@docker
      traefik.http.services.fireshare.loadbalancer.server.port: 80

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to change `AUTH_USERNAME` and `AUTH_PASSWORD` to a custom secret value.

!!! note
    Make sure to change `AUTH_SECRET` to a custom secret value. You can generate this with `openssl rand -hex 32`.

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
