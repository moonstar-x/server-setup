# Media Acquisition

!!! note
    This deprecation page comes from a maintained multi-service stack.

[Jackett](https://github.com/Jackett/Jackett) is a torrent indexer that standardizes the shape of the data from multiple indexers to make it possible to use originally unsupported indexers on services like *Sonarr*. There is no official docker image for *Jackett*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/jackett/).

!!! warning "Deprecation Warning"
    This service was deprecated on the server in favor of [Prowlarr](../../services/media/media-acquisition.md) for its better integration with -arr software.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/media_acquisition
```

We'll need to download a GUI for Transmission manually. For this, inside the newly created folder, run the following commands:

```bash
curl -OL https://github.com/johman10/flood-for-transmission/releases/download/latest/flood-for-transmission.zip
unzip flood-for-transmission.zip && rm flood-for-transmission.zip
mv flood-for-transmission transmission-web-ui
```

## Docker Compose

*Media Acquisition* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  jackett:
    image: ghcr.io/linuxserver/jackett:latest
    restart: unless-stopped
    volumes:
      - ./jackett-config:/config
    ports:
      - 9117:9117
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil
      - AUTO_UPDATE=true
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's web UI port on our firewall.

```bash
sudo ufw allow 9117/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
