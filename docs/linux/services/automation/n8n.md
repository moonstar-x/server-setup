# n8n

[n8n](https://n8n.io/) is a self-hosted node based automation to run run jobs based on triggers similarly to IFTTT.

There is an official image for this service that we'll use: [n8nio/n8n](https://hub.docker.com/r/n8nio/n8n).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/automation/n8n
```

## Docker Compose

*n8n* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: n8nio/n8n:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - n8n
    volumes:
      - ./data:/home/node/.n8n
    environment:
      TZ: America/Guayaquil
      GENERIC_TIMEZONE: America/Guayaquil
      NODE_ENV: production
      N8N_PORT: 5678
      N8N_PROTOCOL: https
      N8N_HOST: subdomain.example.com
      WEBHOOK_URL: https://subdomain.example.com/
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.n8n.rule: Host(`subdomain.example.com`)
      traefik.http.routers.n8n.entrypoints: public
      traefik.http.routers.n8n.service: n8n@docker
      traefik.http.services.n8n.loadbalancer.server.port: 5678

  mongo:
    image: mongo:latest
    restart: unless-stopped
    volumes:
      - ./mongo:/data/db
    environment:
      TZ: America/Guayaquil
      MONGO_INITDB_ROOT_USERNAME: DATABASE_USER
      MONGO_INITDB_ROOT_PASSWORD: DATABASE_PASSWORD

  mongo-express:
    image: mongo-express:latest
    restart: unless-stopped
    depends_on:
      - mongo
    networks:
      default:
      proxy_external:
        aliases:
          - automation-mongo
    environment:
      TZ: America/Guayaquil
      ME_CONFIG_OPTIONS_EDITORTHEME: ambiance
      ME_CONFIG_BASICAUTH_USERNAME: BASIC_AUTH_USER
      ME_CONFIG_BASICAUTH_PASSWORD: BASIC_AUTH_PASSWORD
      ME_CONFIG_MONGODB_SERVER: mongo
      ME_CONFIG_MONGODB_ADMINUSERNAME: DATABASE_USER
      ME_CONFIG_MONGODB_ADMINPASSWORD: DATABASE_PASSWORD
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.automation-mongo.rule: Host(`mongo.automation.home.arpa`)
      traefik.http.routers.automation-mongo.entrypoints: local
      traefik.http.routers.automation-mongo.service: automation-mongo@docker
      traefik.http.services.automation-mongo.loadbalancer.server.port: 8081

networks:
  proxy_external:
    external: true
```

!!! note
    Make sure to change `DATABASE_USER` and `DATABASE_PASSWORD` to a custom secret value.

!!! note
    Make sure to change `BASIC_AUTH_USER` and `BASIC_AUTH_PASSWORD` to a custom secret value.

!!! note
    Replace `subdomain.example.com` with the domain name where your service will be accessible from.

We also need a database for workflows that require data persistence, in this case we've chosen `mongo` but any database that is supported by `n8n` can work just fine.

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