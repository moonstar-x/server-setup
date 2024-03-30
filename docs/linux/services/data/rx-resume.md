# Reactive Resume

[Reactive Resume](https://rxresu.me/) is a tool to easily create beautiful CVs.

There is an official image for this service that we'll use: [amruthpillai/reactive-resume](https://hub.docker.com/r/amruthpillai/reactive-resume).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/rxresume
```

## Docker Compose

*Reactive Resume* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  db:
    image: postgres:15-alpine
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      TZ: America/Guayaquil
      POSTGRES_DB: rxresume
      POSTGRES_USER: rxresume
      POSTGRES_PASSWORD: DATABASE_PASSWORD

  server:
    image: amruthpillai/reactive-resume:server-latest
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - 42000:3100
    environment:
      TZ: America/Guayaquil
      PUBLIC_URL: http://public_client_domain.com
      PUBLIC_SERVER_URL: http://public_api_domain.com
      POSTGRES_DB: rxresume
      POSTGRES_USER: rxresume
      POSTGRES_PASSWORD: DATABASE_PASSWORD
      SECRET_KEY: SECRET_KEY
      POSTGRES_HOST: db
      POSTGRES_PORT: 5432
      JWT_SECRET: JWT_SECRET
      JWT_EXPIRY_TIME: 604800

  client:
    image: amruthpillai/reactive-resume:client-latest
    restart: unless-stopped
    depends_on:
      - server
    ports:
      - 42010:3000
    environment:
      TZ: America/Guayaquil
      PUBLIC_URL: http://public_client_domain.com
      PUBLIC_SERVER_URL: http://public_api_domain.com
```

!!! note
    Make sure to change `DATABASE_PASSWORD`, `SECRET_KEY`, and `JWT_SECRET` to a custom secret value.

!!! note
    Make sure to change `http://public_client_domain.com` and `http://public_api_domain.com` to the domains where each service is hosted.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
