# Reactive Resume

[Reactive Resume](https://rxresu.me/) is a tool to easily create beautiful CVs. This service has an official image available on [Docker Hub](https://hub.docker.com/r/amruthpillai/reactive-resume) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the other server's data will be saved.

```bash
mkdir ~/other/rxresume
```

## Docker Compose

*Reactive Resume* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  db:
    image: postgres:alpine
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/postgresql/data
    environment:
      - TZ=America/Guayaquil
      - POSTGRES_DB=rxresume
      - POSTGRES_USER=rxresume
      - POSTGRES_PASSWORD=CHANGE_THIS_DB_PASSWORD

  server:
    image: amruthpillai/reactive-resume:server-latest
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - 40100:3100
    environment:
      - TZ=America/Guayaquil
      - PUBLIC_URL=http://CHANGE_THIS_IP_OR_DOMAIN:40000
      - PUBLIC_SERVER_URL=http://CHANGE_THIS_IP_OR_DOMAIN:40100
      - POSTGRES_DB=rxresume
      - POSTGRES_USER=rxresume
      - POSTGRES_PASSWORD=CHANGE_THIS_DB_PASSWORD
      - SECRET_KEY=CHANGE_THIS_SECRET
      - POSTGRES_HOST=db
      - POSTGRES_PORT=5432
      - JWT_SECRET=CHANGE_THIS_JWT_SECRET
      - JWT_EXPIRY_TIME=604800

  client:
    image: amruthpillai/reactive-resume:client-latest
    restart: unless-stopped
    depends_on:
      - server
    ports:
      - 40000:3000
    environment:
      - TZ=America/Guayaquil
      - PUBLIC_URL=http://CHANGE_THIS_IP_OR_DOMAIN:40000
      - PUBLIC_SERVER_URL=http://CHANGE_THIS_IP_OR_DOMAIN:40100
```

!!! note
    Make sure to change all the environment variables that begin with `CHANGE_THIS` in your own configuration.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 40000/tcp
sudo ufw allow 40100/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
