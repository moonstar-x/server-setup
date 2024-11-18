# LinkStack

[LinkStack](https://linkstack.org) is a "link in bio" tool similar to LinkTr.ee.

There is an official image for this service that we'll use: [linkstackorg/linkstack](https://hub.docker.com/r/linkstackorg/linkstack).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/linkstack
```

## Docker Compose

*LinkStack* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: linkstackorg/linkstack:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - linkstack
    volumes:
      - linkstack_data:/htdocs
    environment:
      TZ: America/Guayaquil
      SERVER_ADMIN: YOUR_EMAIL_HERE
      HTTP_SERVER_NAME: subdomain.example.com
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.linkstack.rule: Host(`subdomain.example.com`)
      traefik.http.routers.linkstack.entrypoints: tunnel
      traefik.http.routers.linkstack.service: linkstack@docker
      traefik.http.services.linkstack.loadbalancer.server.port: 80

networks:
  proxy_external:
    external: true

volumes:
  linkstack_data:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ./data
```

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

!!! note
    Replace `YOUR_EMAIL_HERE` with your actual email.

!!! note
    This service requires a volume to be added with a bind mount instead of a direct filesystem volume.

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
