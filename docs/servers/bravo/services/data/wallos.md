# Wallos

[Wallos](https://wallosapp.com) is a tracker for subscriptions. It serves as a way to take note of the subscriptions you have and get a monthly and yearly view of the total cost.

There is an official image for this service that we'll use: [bellamy/wallos](https://hub.docker.com/r/bellamy/wallos).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/wallos
```

## Docker Compose

*Wallos* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: bellamy/wallos:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - wallos
    volumes:
      - ./data:/var/www/html/db
      - ./logos:/var/www/html/images/uploads/logos
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.wallos.rule: Host(`subdomain.example.com`)
      traefik.http.routers.wallos.entrypoints: tunnel
      traefik.http.routers.wallos.service: wallos@docker
      traefik.http.services.wallos.loadbalancer.server.port: 80

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
