# Code Server

[Code Server](https://coder.com/) is a web based VSCode UI that allows you to run your projects from within the server. We'll use it in this case to manage the docker compose stacks through this.

There is an official image for this service that we'll use: [ghcr.io/linuxserver/code-server](https://hub.docker.com/r/linuxserver/code-server).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/code-server
```

## Docker Compose

*Code Server* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/linuxserver/code-server:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - code-server
    volumes:
      - ./config:/config
      - ~/:/config/workspace
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
      PASSWORD: '031688'
      PROXY_DOMAIN: code.home.arpa
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.code-server.rule: Host(`code.home.arpa`)
      traefik.http.routers.code-server.entrypoints: local
      traefik.http.routers.code-server.service: code-server@docker
      traefik.http.services.code-server.loadbalancer.server.port: 8443

networks:
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
