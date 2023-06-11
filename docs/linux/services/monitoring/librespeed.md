# LibreSpeed

[LibreSpeed](https://github.com/librespeed/speedtest) is a self-hosted speed test, useful to check our connection from outside into our server. There is no official docker image for *LibreSpeed*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/librespeed).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/monitoring/librespeed
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  librespeed:
    image: ghcr.io/linuxserver/librespeed:latest
    restart: unless-stopped
    volumes:
      - ./config:/config
    ports:
      - 8050:80
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil
      - PASSWORD=CHANGE_THIS
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8050/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
