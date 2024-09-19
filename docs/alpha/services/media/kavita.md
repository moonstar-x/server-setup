# Kavita

[Kavita](https://www.kavitareader.com/) is an eBook server for PDFs and EPUBs.

There is an official image for this service that we'll use: [jvmilazz0/kavita](https://hub.docker.com/r/jvmilazz0/kavita).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/kavita
```

## Docker Compose

*Kavita* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: jvmilazz0/kavita:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - kavita
    volumes:
      - ./data:/kavita/config
      - /media/sata_2tb/Books:/books
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.kavita.rule: Host(`subdomain.example.com`)
      traefik.http.routers.kavita.entrypoints: public
      traefik.http.routers.kavita.service: kavita@docker
      traefik.http.services.kavita.loadbalancer.server.port: 5000

networks:
  proxy_external:
    external: true
```

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
