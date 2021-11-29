# Scrutiny

[Scrutiny](https://github.com/AnalogJ/scrutiny) is a S.M.A.R.T monitoring tool that uses `smartd`. There is no official docker image for *Scrutiny*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/scrutiny).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/monitoring/scrutiny
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  scrutiny:
    image: ghcr.io/linuxserver/scrutiny:latest
    restart: unless-stopped
    cap_add:
      - SYS_RAWIO
    volumes:
      - ./config:/config
      - /run/udev:/run/udev:ro
    ports:
      - 8020:8080
    devices:
      - /dev/sda:/dev/sda
      - /dev/sdb:/dev/sdb
      - /dev/sdc:/dev/sdc
      - /dev/sdd:/dev/sdd
      - /dev/sde:/dev/sde
    environment:
      - PUID=1000
      - PGID=1000
      - SCRUTINY_WEB=true
      - SCRUTINY_COLLECTOR=true
      - TZ=America/Guayaquil
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    Under `devices` make sure you're passing your hard drives. You can check `blkid` to see which device blocks to pass.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8020/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.

## Updating Results

To manually update the results, run the following from inside the container's data folder:

```bash
docker-compose run --rm scrutiny scrutiny-collector-metrics run
```
