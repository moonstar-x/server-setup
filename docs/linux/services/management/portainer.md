# Portainer

[Portainer](https://www.portainer.io/) is a web UI for Docker which allows us to have an insight on all the containers running on our server.

There is an official image for this service that we'll use: [portainer/portainer-ce](https://hub.docker.com/r/portainer/portainer-ce).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/management/portainer
```

## Docker Compose

*Portainer* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: portainer/portainer-ce:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - portainer
    ports:
      - 8000:8000
    volumes:
      - ./data:/data
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.portainer.rule: Host(`subdomain.example.com`)
      traefik.http.routers.portainer.entrypoints: public
      traefik.http.routers.portainer.service: portainer@docker
      traefik.http.services.portainer.loadbalancer.server.port: 9000

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
