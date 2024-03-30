# Lidarr

[Lidarr](https://lidarr.audio/) is an RSS downloader focused on music. For this service we will use a fork that comes integrated with [Deemix](https://www.reddit.com/r/deemix/) which allows free download of music from Deezer.

For this service, we'll use [youegraillot/lidarr-on-steroids](https://hub.docker.com/r/youegraillot/lidarr-on-steroids).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/lidarr
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Lidarr* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  lidarr:
    image: youegraillot/lidarr-on-steroids:latest
    restart: unless-stopped
    networks:
      - default
      - downloads_external
    ports:
      - 45400:8686
      - 45410:6595
    volumes:
      - ./config:/config
      - ./deemix:/config_deemix
      - /media/sata_2tb/Plex Music:/music
      - /media/sata_2tb/Downloads:/downloads
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
      AUTOCONFIG: true

networks:
  downloads_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
