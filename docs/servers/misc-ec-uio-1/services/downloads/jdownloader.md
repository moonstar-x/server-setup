# JDownloader

[JDownloader](https://jdownloader.org/) is a download client that makes downloading from direct links a breeze.

There is no official image for this service, so we'll use [jlesage/jdownloader-2](https://hub.docker.com/r/jlesage/jdownloader-2).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/jdownloader
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*JDownloader* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: jlesage/jdownloader-2:latest
    restart: unless-stopped
    networks:
      default:
      downloads_external:
      proxy_external:
        aliases:
          - jdownloader
    ports:
      - 3129:3129
    volumes:
      - ./config:/config
      - /media/sata_2tb/Downloads:/output
    environment:
      TZ: America/Guayaquil
      USER_ID: 1000
      GROUP_ID: 1000
      VNC_PASSWORD: ${BASIC_AUTH_PASSWORD}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.jdownloader.rule: Host(`${DOMAIN_JDOWNLOADER_LOCAL}`) || Host(`${DOMAIN_JDOWNLOADER_VPN}`)
      traefik.http.routers.jdownloader.entrypoints: local-https
      traefik.http.routers.jdownloader.tls: true
      traefik.http.routers.jdownloader.tls.certresolver: le
      traefik.http.routers.jdownloader.service: jdownloader@docker
      traefik.http.services.jdownloader.loadbalancer.server.port: 5800

networks:
  downloads_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    In the case of the `USER_ID` and `GROUP_ID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_JDOWNLOADER_LOCAL=
DOMAIN_JDOWNLOADER_VPN=
BASIC_AUTH_PASSWORD=
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
