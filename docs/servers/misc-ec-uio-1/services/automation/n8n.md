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
    depends_on:
      - mongo
      - redis
      - rabbitmq
    extra_hosts:
      - host.docker.internal:host-gateway
    networks:
      default:
      proxy_external:
        aliases:
          - n8n
    volumes:
      - ./data:/home/node/.n8n
      - ./local:/home/node/host
      - /media/sata_2tb/MVs:/home/node/binds/MVs
    environment:
      TZ: America/Guayaquil
      GENERIC_TIMEZONE: America/Guayaquil
      NODE_ENV: production
      N8N_PORT: 5678
      N8N_PROTOCOL: https
      N8N_HOST: ${DOMAIN_N8N}
      WEBHOOK_URL: https://${DOMAIN_N8N}/
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.n8n.rule: Host(`${DOMAIN_N8N}`)
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
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}

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
      ME_CONFIG_BASICAUTH_USERNAME: ${MONGO_BASIC_AUTH_USER}
      ME_CONFIG_BASICAUTH_PASSWORD: ${MONGO_BASIC_AUTH_PASSWORD}
      ME_CONFIG_MONGODB_SERVER: mongo
      ME_CONFIG_MONGODB_ADMINUSERNAME: ${MONGO_USER}
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_PASSWORD}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.automation-mongo.rule: Host(`${DOMAIN_MONGO_LOCAL}`) || Host(`${DOMAIN_MONGO_VPN}`)
      traefik.http.routers.automation-mongo.entrypoints: local-https
      traefik.http.routers.automation-mongo.tls: true
      traefik.http.routers.automation-mongo.tls.certresolver: le
      traefik.http.routers.automation-mongo.service: automation-mongo@docker
      traefik.http.services.automation-mongo.loadbalancer.server.port: 8081

  redis:
    image: redis:latest
    restart: unless-stopped
    environment:
      TZ: America/Guayaquil

  redis-insight:
    image: redis/redisinsight:latest
    restart: unless-stopped
    user: 1000:1000
    depends_on:
      - redis
    networks:
      default:
      proxy_external:
        aliases:
          - automation-redis
    volumes:
      - ./redis-insight:/data
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.automation-redis.rule: Host(`${DOMAIN_REDIS_LOCAL}`) || Host(`${DOMAIN_REDIS_VPN}`)
      traefik.http.routers.automation-redis.entrypoints: local-https
      traefik.http.routers.automation-redis.tls: true
      traefik.http.routers.automation-redis.tls.certresolver: le
      traefik.http.routers.automation-redis.service: automation-redis@docker
      traefik.http.services.automation-redis.loadbalancer.server.port: 5540

  rabbitmq:
    image: rabbitmq:3-management
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - automation-rabbitmq
    volumes:
      - ./rabbitmq:/var/lib/rabbitmq
    environment:
      TZ: America/Guayaquil
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.automation-rabbit-mq.rule: Host(`${DOMAIN_RABBITMQ_LOCAL}`) || Host(`${DOMAIN_RABBITMQ_VPN}`)
      traefik.http.routers.automation-rabbit-mq.entrypoints: local-https
      traefik.http.routers.automation-rabbit-mq.tls: true
      traefik.http.routers.automation-rabbit-mq.tls.certresolver: le
      traefik.http.routers.automation-rabbit-mq.service: automation-rabbit-mq@docker
      traefik.http.services.automation-rabbit-mq.loadbalancer.server.port: 15672

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_N8N=

DOMAIN_MONGO_LOCAL=
DOMAIN_MONGO_VPN=

DOMAIN_REDIS_LOCAL=
DOMAIN_REDIS_VPN=

DOMAIN_RABBITMQ_LOCAL=
DOMAIN_RABBITMQ_VPN=

MONGO_USER=
MONGO_PASSWORD=
MONGO_BASIC_AUTH_USER=
MONGO_BASIC_AUTH_PASSWORD=

RABBITMQ_USER=
RABBITMQ_PASSWORD=
```

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

### Extras

Technically to get *n8n* up and running you only need the `web` container. However, we've extended this stack with other services to use alongside *n8n* like:

* *mongo* as a database for data persistence.
* *redis* as a cache for caching certain operations to accelerate certain workflows.
* *rabbitmq* as a message queue for processing events for certain workflows.

Each of these services come with web based dashboards to access their data from outside if needed.

#### Local Data

Additionally, you may have noticed the volumes in the `web` container. Only the first volume is required to keep *n8n*'s data, the rest are extra volumes that we've included to interface with the host's local filesystem.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
