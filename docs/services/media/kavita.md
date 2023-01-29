# Kavita

[Kavita](https://www.kavitareader.com/) is an eBook server for PDFs and EPUBs. This service has an official image available on [Docker Hub](https://hub.docker.com/r/kizaing/kavita) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/kavita
```

## Docker Compose

*Kavita* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  kavita:
    image: kizaing/kavita:latest
    restart: unless-stopped
    ports:
      - 5000:5000
    volumes:
      - ./data:/kavita/config
      - /media/sata_2tb/Nextcloud/data/__groupfolders/1:/books
    environment:
      - TZ=America/Guayaquil
```

!!! note
    If you're curious about that weird volume for the books, currently this set up syncs up the books from [Nextcloud](../data/nextcloud.md) through
    the use of a plugin named "Shared Group Folders" which allows multiple users to manage this folder with ease.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 5000/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
