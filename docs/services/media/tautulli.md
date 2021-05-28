# Tautulli

[Tautulli](https://tautulli.com/) is a monitoring tool for *Plex*. This service has an official image available on [Docker Hub](https://hub.docker.com/r/tautulli/tautulli) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/tautulli
```

## Docker Compose

*Tautulli* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  tautulli:
    image: tautulli/tautulli:latest
    restart: unless-stopped
    volumes:
      - ./config:/config
    ports:
      - 8181:8181
    environment:
      - TZ=America/Guayaquil
      - PUID=1000
      - PGID=1000
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8181/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
