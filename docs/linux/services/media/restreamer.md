# Restreamer

[Restreamer](https://docs.datarhei.com/restreamer) allows you to stream any type of media into multiple platforms simultaneously.

There is an official image for this service that we'll use: [datarhei/restreamer](https://hub.docker.com/r/datarhei/restreamer).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/restreamer
```

## Docker Compose

*Restreamer* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: datarhei/restreamer:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - restreamer
    ports:
      - 1935:1935
    volumes:
      - ./config:/core/config
      - ./data:/core/data
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.restreamer.rule: Host(`stream.home.example.com`, `stream.vpn.example.com`)
      traefik.http.routers.restreamer.entrypoints: local-https
      traefik.http.routers.restreamer.tls: true
      traefik.http.routers.restreamer.tls.certresolver: le
      traefik.http.routers.restreamer.service: restreamer@docker
      traefik.http.services.restreamer.loadbalancer.server.port: 8080

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
