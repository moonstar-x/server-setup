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

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
