# Lidarr

[Lidarr](https://lidarr.audio/) is an RSS downloader focused on music. For this service we will use a fork that comes integrated with [Deemix](https://www.reddit.com/r/deemix/) which allows free download of music from Deezer.

For this service, we'll use [youegraillot/lidarr-on-steroids](https://hub.docker.com/r/youegraillot/lidarr-on-steroids).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/lidarr
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Lidarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: youegraillot/lidarr-on-steroids:latest
    restart: unless-stopped
    networks:
      default:
      downloads_external:
      proxy_external:
        aliases:
          - lidarr
    volumes:
      - ./config:/config
      - ./deemix:/config_deemix
      - /media/sata_2tb/Plex Music:/music
      - /media/sata_2tb/Downloads:/downloads
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
      AUTOCONFIG: true
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.lidarr.rule: Host(`lidarr.home.arpa`)
      traefik.http.routers.lidarr.entrypoints: local
      traefik.http.routers.lidarr.service: lidarr@docker
      traefik.http.services.lidarr.loadbalancer.server.port: 8686
      traefik.http.routers.deemix.rule: Host(`deemix.home.arpa`)
      traefik.http.routers.deemix.entrypoints: local
      traefik.http.routers.deemix.service: deemix@docker
      traefik.http.services.deemix.loadbalancer.server.port: 6595

networks:
  downloads_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

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
