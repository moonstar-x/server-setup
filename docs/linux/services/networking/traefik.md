# Traefik

[Portainer](https://traefik.io/traefik/) is a reverse proxy with a first class integration with Docker.

There is an official image for this service that we'll use: [traefik](https://hub.docker.com/r/_/traefik).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/networking/traefik
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `tunnel_external` and `proxy_external` networks before defining the `docker-compose.yml` file. If you haven't created these networks, you can do so with:

```bash
docker network create tunnel_external
docker network create proxy_external
```

## Docker Compose

*Traefik* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  traefik:
    image: traefik:latest
    restart: unless-stopped
    networks:
      - default
      - proxy_external
      - tunnel_external
    ports:
      - 80:8020
      - 9000:8080
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command:
      - '--log.level=DEBUG'
      - '--api.insecure=true'
      - '--providers.docker=true'
      - '--providers.docker.exposedByDefault=false'
      - '--entrypoints.public.address=:8000'
      - '--entrypoints.local.address=:8020'
    environment:
      TZ: America/Guayaquil

networks:
  proxy_external:
    external: true
  tunnel_external:
    external: true
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.

## Further Reading

This page just described the basic steps to follow to set up the reverse proxy. This is far from done, make sure you check out:

* How to expose web services via Cloudflare with [cloudflared](./cloudflared.md).
* How to set forward authentication with [dex](./dex.md).
