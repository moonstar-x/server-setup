# Jellyfin + EmbyStat

!!! note
    This is a multi-service stack.

[Jellyfin](https://jellyfin.org/) is an open source version of the famous media server *Emby*. This media server has an official image available on [Docker Hub](https://hub.docker.com/r/jellyfin/jellyfin) which we'll use.

[EmbyStat](https://github.com/mregni/EmbyStat) is an analytics service for Jellyfin/Emby. There is no official docker image for *EmbyStat*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/embystat).

### Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/jellyfin
```

## Docker Compose

*Jellyfin + EmbyStat* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    user: 1000:1000
    restart: unless-stopped
    volumes:
      - ./jellyfin-config:/config
      - ./jellyfin-cache:/cache
      - /media/usb_4tb_2/Jellyfin:/media
    ports:
      - 8096:8096
    environment:
      - TZ=America/Guayaquil

  embystat:
    image: ghcr.io/linuxserver/embystat:latest
    restart: unless-stopped
    depends_on:
      - jellyfin
    volumes:
      - ./embystat-config:/config
    ports:
      - 6555:6555
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil
```

!!! note
    In the case of the `user` directive, `1000:1000` corresponds to the user's `UID:GID`. You can find the values for your own user by running `id $whoami`.

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8086/tcp
sudo ufw allow 6555/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
