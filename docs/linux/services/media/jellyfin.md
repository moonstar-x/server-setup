# Jellyfin

[Jellyfin](https://jellyfin.org/) is an open source version of the famous media server *Emby*.

There is an official image for this service that we'll use: [jellyfin/jellyfin](https://hub.docker.com/r/jellyfin/jellyfin).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/media/jellyfin
```

## Docker Compose

*Jellyfin* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: jellyfin/jellyfin:latest
    restart: unless-stopped
    user: 1000:1000
    networks:
      default:
      proxy_external:
        aliases:
          - jellyfin
    volumes:
      - ./jellyfin-config:/config
      - ./jellyfin-cache:/cache
      - /media/usb_4tb_2/Jellyfin:/media
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.jellyfin.rule: Host(`jellyfin.home.example.com`) || Host(`jellyfin.vpn.example.com`)
      traefik.http.routers.jellyfin.entrypoints: local-https
      traefik.http.routers.jellyfin.tls: true
      traefik.http.routers.jellyfin.tls.certresolver: le
      traefik.http.routers.jellyfin.service: jellyfin@docker
      traefik.http.services.jellyfin.loadbalancer.server.port: 8096

networks:
  proxy_external:
    external: true
```

!!! note
    In the case of the `user` directive, `1000:1000` corresponds to the user's `UID:GID`. You can find the values for your own user by running `id $whoami`.

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
