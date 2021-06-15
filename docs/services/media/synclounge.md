# Synclounge

[Synclounge](https://synclounge.tv/) is a tool that lets your Plex users watch something simultaneously. There is no official docker image for *Synclounge*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/synclounge).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/synclounge
```

## Docker Compose

*Synclounge* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  synclounge:
    image: ghcr.io/linuxserver/synclounge:latest
    restart: unless-stopped
    ports:
      - 8088:8088
    environment:
      - TZ=America/Guayaquil
```

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8088/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
