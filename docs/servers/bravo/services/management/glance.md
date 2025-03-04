# Glance

[Glance](https://github.com/glanceapp/glance) is a homepage builder with plenty of integrations.

There is an official image for this service that we'll use: [glanceapp/glance](https://hub.docker.com/r/glanceapp/glance).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/management/glance
```

## Docker Compose

*Glance* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: glanceapp/glance:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - glance
    volumes:
      - ./config:/app/config
      - ./assets:/app/assets
    environment:
      TZ: America/Guayaquil
      GITHUB_TOKEN: GITHUB_TOKEN_HERE
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.glance.rule: Host(`subdomain.example.com`)
      traefik.http.routers.glance.entrypoints: tunnel
      traefik.http.routers.glance.service: glance@docker
      traefik.http.services.glance.loadbalancer.server.port: 8080

networks:
  proxy_external:
    external: true
```

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

!!! note
    You may not need to pass a `GITHUB_TOKEN` variable. I have added one because my [configuration](https://github.com/glanceapp/glance/blob/main/docs/configuration.md#token) benefits from using it.

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Post Configuration

To configure this, make sure to create a `config/glance.yml` file. Check out the [documentation](https://github.com/glanceapp/glance/blob/main/docs/configuration.md) to see what you can add.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
