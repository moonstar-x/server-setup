# Sonarqube

[Sonarqube](https://www.sonarsource.com/es/products/sonarqube/) is a self-hosted code quality and static analysis tool.

There is an official image for this service that we'll use: [sonarqube](https://hub.docker.com/r/_/sonarqube/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/sonarqube
```

## Docker Compose

*Sonarqube* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: sonarqube:community
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    networks:
      default:
      proxy_external:
        aliases:
          - sonarqube
    volumes:
      - ./sonarqube/data:/opt/sonarqube/data
      - ./sonarqube/extensions:/opt/sonarqube/extensions
      - ./sonarqube/logs:/opt/sonarqube/logs
      - ./sonarqube/temp:/opt/sonarqube/temp
    environment:
      TZ: America/Guayaquil
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/${POSTGRES_DATABASE}
      SONAR_JDBC_USERNAME: ${POSTGRES_USER}
      SONAR_JDBC_PASSWORD: ${POSTGRES_PASSWORD}
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.sonarqube.rule: Host(`${DOMAIN_SONARQUBE}`)
      traefik.http.routers.sonarqube.entrypoints: public
      traefik.http.routers.sonarqube.service: sonarqube@docker
      traefik.http.services.sonarqube.loadbalancer.server.port: 9000

  db:
    image: postgres:17
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: ${POSTGRES_DATABASE}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
POSTGRES_DATABASE=
POSTGRES_USER=
POSTGRES_PASSWORD=

DOMAIN_SONARQUBE=
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
