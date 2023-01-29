# Nextcloud

!!! warning
    This section contains information of a service that has been deprecated. The server no longer runs this service. This page will no longer be updated.

[Nextcloud](https://nextcloud.com/) is a self-hosted cloud data server, useful for keeping documents in your own server. This service has an official image on [Docker Hub](https://hub.docker.com/_/nextcloud) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/nextcloud
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  nextcloud:
    image: nextcloud:latest
    restart: unless-stopped
    volumes:
      - ./data:/var/www/html
    ports:
      - 9020:80
    depends_on:
      - db
    environment:
      - TZ=America/Guayaquil
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=CHANGE_THIS
      - MYSQL_HOST=db

  db:
    image: mariadb:10
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=CHANGE_THIS
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=CHANGE_THIS
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 9020/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
