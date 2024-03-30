# LibreSpeed

[LibreSpeed](https://github.com/librespeed/speedtest) is a self-hosted speed test, useful to check our connection from outside into our server.

There is no official image for this service, so we'll use [ghcr.io/linuxserver/librespeed](https://hub.docker.com/r/linuxserver/librespeed).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/observability/librespeed
```

## Docker Compose

*LibreSpeed* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  librespeed:
    image: ghcr.io/linuxserver/librespeed:latest
    restart: unless-stopped
    ports:
      - 22000:80
    volumes:
      - ./config:/config
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
      PASSWORD: DATABASE_PASSWORD
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    Make sure to change `DATABASE_PASSWORD` to a custom value.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
