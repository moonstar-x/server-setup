# Traefik

[Traefik](https://traefik.io/traefik/) is a reverse proxy with a first-class integration with Docker.

There is an official image for this service that we'll use: [traefik](https://hub.docker.com/r/_/traefik).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/networking/traefik
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `proxy_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create proxy_external
```

## Configuration

Create a `traefik.yml` file with the following content:

```yaml
global:
  checkNewVersion: true

log:
  level: DEBUG

api:
  insecure: true
  dashboard: true

providers:
  docker:
    exposedByDefault: false
    watch: true
  file:
    filename: /etc/traefik/traefik.yaml
    watch: true

entryPoints:
  http:
    address: :80
  https:
    address: :443

certificatesresolvers:
  le:
    acme:
      httpChallenge:
        entryPoint: http
      email: YOUR_EMAIL_HERE
      storage: /letsencrypt/acme.json
```

!!! note
    Make sure to replace `YOUR_EMAIL_HERE` with your actual email.

## Docker Compose

*Traefik* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  proxy:
    image: traefik:latest
    restart: unless-stopped
    extra_hosts:
      - host.docker.internal:host-gateway
    networks:
      default:
      proxy_external:
        aliases:
          - traefik
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yaml
      - ./letsencrypt:/letsencrypt
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.traefik-http.rule: Host(`${DOMAIN_TRAEFIK}`)
      traefik.http.routers.traefik-http.entrypoints: http
      traefik.http.routers.traefik-http.middlewares: traefik-redirectscheme
      traefik.http.routers.traefik-http.service: traefik@docker
      traefik.http.routers.traefik-https.rule: Host(`${DOMAIN_TRAEFIK}`)
      traefik.http.routers.traefik-https.entrypoints: https
      traefik.http.routers.traefik-https.service: traefik@docker
      traefik.http.routers.traefik-https.tls: true
      traefik.http.routers.traefik-https.tls.certresolver: le
      traefik.http.services.traefik.loadbalancer.server.port: 8080
      traefik.http.middlewares.traefik-redirectscheme.redirectscheme.scheme: https
      traefik.http.middlewares.traefik-redirectscheme.redirectscheme.permanent: true

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_TRAEFIK=
```

### Reverse Proxy

*Traefik* usually comes with a web dashboard for managing the resources exposed. Now, we're actually using it to expose its dashboard itself.

For this reason, you will see that this service has:

1. A number of labels with names starting with `traefik`.

If you don't want to use the proxy itself to expose its own dashboard, feel free to remove those labels and bind the dashboard port manually.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
