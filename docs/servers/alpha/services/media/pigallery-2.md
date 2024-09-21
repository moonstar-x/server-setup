# PiGallery 2

[PiGallery 2](https://bpatrik.github.io/pigallery2/) is a simple image gallery.

There is an official image for this service that we'll use: [bpatrik/pigallery2](https://hub.docker.com/r/bpatrik/pigallery2).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/pigallery2
```

## Docker Compose

*PiGallery 2* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: bpatrik/pigallery2:1.9.5
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - pigallery
    volumes:
      - ./config:/app/data/config
      - ./data:/app/data/db
      - /media/usb_4tb_2/Gallery/Caches:/app/data/tmp
      - /media/usb_4tb_2/Gallery/Images:/app/data/images:ro
    environment:
      TZ: America/Guayaquil
      NODE_ENV: production
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.pigallery.rule: Host(`gallery.alpha.example.com`) || Host(`gallery.alpha.home.example.com`)
      traefik.http.routers.pigallery.entrypoints: local-https
      traefik.http.routers.pigallery.tls: true
      traefik.http.routers.pigallery.tls.certresolver: le
      traefik.http.routers.pigallery.service: pigallery@docker
      traefik.http.services.pigallery.loadbalancer.server.port: 80

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
