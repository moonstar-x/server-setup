# LibreSpeed

[LibreSpeed](https://github.com/librespeed/speedtest) is a self-hosted speed test, useful to check our connection from outside into our server.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/librespeed](https://hub.docker.com/r/linuxserver/librespeed).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/observability/librespeed
```

## Docker Compose

*LibreSpeed* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/librespeed:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - librespeed
    volumes:
      - ./config:/config
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
      PASSWORD: DATABASE_PASSWORD
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.librespeed.rule: Host(`subdomain.example.com`)
      traefik.http.routers.librespeed.entrypoints: public
      traefik.http.routers.librespeed.service: librespeed@docker
      traefik.http.services.librespeed.loadbalancer.server.port: 80

networks:
  proxy_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    Make sure to change `DATABASE_PASSWORD` to a custom value.

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
