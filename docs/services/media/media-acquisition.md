# Media Acquisition

!!! note
    This is a multi-service stack.

[Transmission](https://transmissionbt.com/) is a BitTorrent client. There is no official docker image for *Transmission*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/transmission).

[JDownloader](https://jdownloader.org/) is a download client that makes downloading from direct links a breeze. There is no official image for *JDownloader*, however we'll use one available on [Docker Hub](https://hub.docker.com/r/jlesage/jdownloader-2).

[Sonarr](https://sonarr.tv/) is an RSS downloader focused on TV Shows. There is no official docker image for *Sonarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/sonarr).

[Radarr](https://radarr.video/) is an RSS downloader focused on movies. There is no official docker image for *Radarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/radarr).

[Jackett](https://github.com/Jackett/Jackett) is a torrent indexer that standardizes the shape of the data from multiple indexers to make it possible to use originally unsupported indexers on services like *Sonarr*. There is no official docker image for *Jackett*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/jackett/).

[Ombi](https://ombi.io/) is a media request tracker, useful for when you share a Plex server or similar with family and friends. There is no official docker image for *Ombi*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/ombi).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/media/media_acquisition
```

## Docker Compose

*Media Acquisition* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  transmission:
    image: ghcr.io/linuxserver/transmission:latest
    restart: unless-stopped
    volumes:
      - ./transmission-config:/config
      - ./transmission-watch:/watch
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

  jdownloader:
    image: jlesage/jdownloader-2:latest
    restart: unless-stopped
    volumes:
      - ./jdownloader-config:/config
      - /media/sata_2tb/Downloads:/output
    ports:
      - 3129:3129
      - 5800:5800
    environment:
      - TZ=America/Guayaquil
      - USER_ID=1000
      - GROUP_ID=1000

  sonarr:
    image: ghcr.io/linuxserver/sonarr:latest
    restart: unless-stopped
    volumes:
      - ./sonarr-config:/config
      - /media/usb_1tb:/media/usb_1tb
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
      - /media/sata_2tb/Downloads:/downloads
    ports:
      - 8989:8989
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

  radarr:
    image: ghcr.io/linuxserver/radarr:latest
    restart: unless-stopped
    volumes:
      - ./radarr-config:/config
      - /media/usb_1tb:/media/usb_1tb
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
      - /media/sata_2tb/Downloads:/downloads
    ports:
      - 7878:7878
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

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

  ombi:
    image: ghcr.io/linuxserver/ombi:latest
    restart: unless-stopped
    volumes:
      - ./ombi-config:/config
    ports:
      - 3579:3579
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    The password should be set inside the `docker-compose.yml` file and not manually updated in `transmission-config/settings.json` which will mess up with the s6 supervisor. Use a password you don't care too much about since it would basically be saved in plain text.

!!! note
    You may change the contents of `transmission-config/settings.json` as long as the container is stopped.

!!! note
    In the case of the `USER_ID` and `GROUP_ID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's web UI port on our firewall.

```bash
sudo ufw allow 9091/tcp
sudo ufw allow 3129/tcp
sudo ufw allow 5800/tcp
sudo ufw allow 8989/tcp
sudo ufw allow 7878/tcp
sudo ufw allow 9117/tcp
sudo ufw allow 3579/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
