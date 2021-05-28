# Transmission

[Tranmission](https://transmissionbt.com/) is a BitTorrent client. There is no official docker image for *Transmission*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/transmission).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/transmission
```

We'll also need to create a custom network to allow other containers to communicate with this one.

```bash
docker network create downloader
```

Any container that wants to communicate with this one should join the `downloader` network.

## Docker Compose

*Transmission* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  transmission:
    image: ghcr.io/linuxserver/transmission:latest
    restart: unless-stopped
    networks:
      - downloader
    volumes:
      - ./config:/config
      - ./watch:/watch
      - /media/sata_2tb/Downloads:/downloads
    ports:
      - 9091:9091
      - 51413:51413
      - 51413:51413/udp
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil
      - TRANSMISSION_WEB_HOME=/flood-for-transmission/
      - USER=<USER_HERE>
      - PASS=<PASS_HERE>

networks:
  downloader:
    external: true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    The password should be set inside the `docker-compose.yml` file and not manually updated in `config/settings.json` which will mess up with the s6 supervisor. Use a password you don't care too much about since it would basically be saved in plain text.

## Post-Installation

We'll need to allow the service's web UI port on our firewall.

```bash
sudo ufw allow 9091/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
