# Traefik

[Traefik](https://traefik.io/traefik/) is a reverse proxy with a first class integration with Docker.

There is an official image for this service that we'll use: [traefik](https://hub.docker.com/r/_/traefik).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/networking/traefik
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `tunnel_external` and `proxy_external` networks before defining the `docker-compose.yml` file. If you haven't created these networks, you can do so with:

```bash
docker network create tunnel_external
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
    fileName: /etc/traefik/traefik.yaml
    watch: true

entryPoints:
  tunnel:
    address: :8000
  direct:
    address: :8020

certificatesresolvers:
  le:
    acme:
      dnschallenge:
        provider: cloudflare
        delaybeforecheck: 0
        resolvers: 1.1.1.1
      email: YOUR_EMAIL_HERE
      storage: /letsencrypt/acme.json
```

!!! note
    Make sure to replace `YOUR_EMAIL_HERE` with your actual email.

As a side note, we're using TLS in certain services that point to local IP addresses. I decided to go this route to keep using my own existing domain while keeping HTTPS active. We're using a DNS challenge which requires you to provide a Cloudflare API token with edit access for your DNS zones.

If it does not apply to you, you may want to explore different challenges such as HTTP or TLS, or just serve your content through HTTP altogether.

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
      tunnel_external:
        aliases:
          - traefik
      proxy_external:
        aliases:
          - traefik
    ports:
      - 80:8000
      - 443:8020
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yaml
      - ./letsencrypt:/letsencrypt
    environment:
      TZ: America/Guayaquil
      CF_DNS_API_TOKEN: CLOUDFLARE_API_TOKEN
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.traefik.rule: Host(`proxy.vps.example.com`)
      traefik.http.routers.traefik.entrypoints: direct
      traefik.http.routers.traefik.tls: true
      traefik.http.routers.traefik.tls.certresolver: le
      traefik.http.routers.traefik.service: traefik@docker
      traefik.http.services.traefik.loadbalancer.server.port: 8080

networks:
  proxy_external:
    external: true
  tunnel_external:
    external: true
```

!!! note
    Make sure to replace `CLOUDFLARE_API_TOKEN` with a token you have acquired from your own [dashboard](https://dash.cloudflare.com/).

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

## Further Reading

This page just described the basic steps to follow to set up the reverse proxy. This is far from done, make sure you check out:

* How to expose web services via Cloudflare with [cloudflared](./cloudflared.md).
