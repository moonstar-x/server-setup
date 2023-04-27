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
      - /media/sata_2tb/Books:/books
    environment:
      - TZ=America/Guayaquil
```

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
