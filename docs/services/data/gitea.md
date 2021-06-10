# Gitea

[Gitea](https://gitea.io/en-us/) is a self-hosted git server, useful for having a private VCS solution. This service has an official image on [Docker Hub](https://hub.docker.com/r/gitea/gitea/) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/gitea
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  gitea:
    image: gitea/gitea:latest
    restart: unless-stopped
    volumes:
      - ./data:/data
    ports:
      - 3000:3000
    depends_on:
      - db
    environment:
      - TZ=America/Guayaquil

  db:
    image: mariadb:10
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=CHANGE_THIS
      - MYSQL_DATABASE=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=CHANGE_THIS
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 3000/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
