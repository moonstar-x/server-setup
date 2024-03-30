# Transmission

[Transmission](https://transmissionbt.com/) is a BitTorrent client.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/transmission](https://hub.docker.com/r/linuxserver/transmission).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/downloads/transmission
```

We'll need to download a GUI for Transmission manually. For this, inside the newly created folder, run the following commands:

```bash
curl -OL https://github.com/johman10/flood-for-transmission/releases/download/latest/flood-for-transmission.zip
unzip flood-for-transmission.zip && rm flood-for-transmission.zip
mv flood-for-transmission web-ui
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `downloads_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create downloads_external
```

## Docker Compose

*Transmission* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  transmission:
    image: ghcr.io/linuxserver/transmission:latest
    restart: unless-stopped
    networks:
      - default
      - downloads_external
    ports:
      - 45000:9091
      - 51413:51413
      - 51413:51413/udp
    volumes:
      - ./config:/config
      - ./watch:/watch
      - ./web-ui:/flood-for-transmission
      - /media/sata_2tb/Downloads:/downloads
    environment:
      TZ: America/Guayaquil
      TRANSMISSION_WEB_HOME: /flood-for-transmission/
      USER: BASIC_AUTH_USER
      PASS: BASIC_AUTH_PASSWORD
      PUID: 1000
      PGID: 1000

networks:
  downloads_external:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    Make sure to replace `BASIC_AUTH_USER` and `BASIC_AUTH_PASSWORD` to something secret of your own choosing.

!!! note
    You may change the contents of `transmission-config/settings.json` as long as the container is stopped.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
