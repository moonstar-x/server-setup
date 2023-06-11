# Nextcloud

[Nextcloud](https://nextcloud.com/) is a self-hosted cloud data server, useful for keeping documents in your own server. This service has an official image on [Docker Hub](https://hub.docker.com/_/nextcloud) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/nextcloud
```

## Dockerfile

Despite the service having an official image available, it lacks a dependency necessary to allow for SMB shares as external storage.

The following is the (tiny) `Dockerfile` that will be used for this service:

```docker
FROM nextcloud:stable

RUN apt-get update && apt-get install -y procps smbclient && rm -rf /var/lib/apt/lists/*
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  nextcloud:
    build: .
    restart: unless-stopped
    volumes:
      - /media/sata_2tb/Nextcloud:/var/www/html
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
    image: mariadb:10.5
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=CHANGE_THIS
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=CHANGE_THIS

  cron:
    image: nextcloud:stable
    restart: unless-stopped
    depends_on:
      - nextcloud
    volumes:
      - /media/sata_2tb/Nextcloud:/var/www/html
    entrypoint: /cron.sh
    environment:
      - TZ=America/Guayaquil
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 9020/tcp
```

We'll also need to add the user to the `www-data` group to allow access to the data being uploaded inside the service:

```bash
sudo usermod -aG www-data $USER
```

Additionally, you may need to change the `/media/sata_2tb/Nextcloud/config/config.php` file. Make sure to have the correct array of `trusted_domains`, to
update the `overwrite.cli.url` to the correct URL where the service will be accessible from, and include the following line:

```php
'overwriteprotocol' => 'https',
```

Inside Nextcloud, some applications should be installed, notably the Group Shared Folder one, which allows to quickly share folders with multiple users internally.

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
