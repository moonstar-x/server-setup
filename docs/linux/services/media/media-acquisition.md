# Media Acquisition

!!! note
    This is a multi-service stack.

[Transmission](https://transmissionbt.com/) is a BitTorrent client. There is no official docker image for *Transmission*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/transmission).

[JDownloader](https://jdownloader.org/) is a download client that makes downloading from direct links a breeze. There is no official image for *JDownloader*, however we'll use one available on [Docker Hub](https://hub.docker.com/r/jlesage/jdownloader-2).

[Ombi](https://ombi.io/) is a media request tracker, useful for when you share a Plex server or similar with family and friends. There is no official docker image for *Ombi*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/ombi).

[Sonarr](https://sonarr.tv/) is an RSS downloader focused on TV Shows. There is no official docker image for *Sonarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/sonarr).

[Radarr](https://radarr.video/) is an RSS downloader focused on movies. There is no official docker image for *Radarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/radarr).

[Lidarr](https://lidarr.audio/) is an RSS downloader focused on music. For this service we will use a fork that comes integrated with [Deemix](https://www.reddit.com/r/deemix/) which allows free download of music from Deezer. The image is available on [Docker Hub](https://hub.docker.com/r/youegraillot/lidarr-on-steroids).

[Bazarr](https://www.bazarr.media/) is an RSS downloader focused on subtitles. There is no official docker image for *Bazarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/bazarr).

[Readarr](https://readarr.com/) is an RSS downloader focused on ebooks and audiobooks. There is no official docker image for *Readarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/readarr).

[Prowlarr](https://prowlarr.com/) is an indexer for -arr software. There is no official docker image for *Prowlarr*, however we'll use one from **LinuxServer** on [Docker Hub](https://hub.docker.com/r/linuxserver/prowlarr).

[OpenBooks](https://evan-buss.github.io/openbooks/) is an ebook downloader that can download from IRC Highway. There is an official docker image for *OpenBooks* [GitHub Packages](https://github.com/evan-buss/openbooks/pkgs/container/openbooks) which we'll use.

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
  transmission:
    image: ghcr.io/linuxserver/transmission:latest
    restart: unless-stopped
    volumes:
      - ./transmission-config:/config
      - ./transmission-watch:/watch
      - ./transmission-web-ui:/flood-for-transmission
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
      - VNC_PASSWORD=<PASS_HERE>

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

  sonarr:
    image: ghcr.io/linuxserver/sonarr:latest
    restart: unless-stopped
    volumes:
      - ./sonarr-config:/config
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
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
      - /media/sata_2tb/Downloads:/downloads
    ports:
      - 7878:7878
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

  lidarr:
    image: youegraillot/lidarr-on-steroids:latest
    restart: unless-stopped
    volumes:
      - ./lidarr-config:/config
      - ./deemix-config:/config_deemix
      - /media/sata_2tb/Plex Music:/music
      - /media/sata_2tb/Deemix Downloads:/downloads
    ports:
      - 8686:8686
      - 6595:6595
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil
      - AUTOCONFIG=true

  bazarr:
    image: ghcr.io/linuxserver/bazarr:latest
    restart: unless-stopped
    volumes:
      - ./bazarr-config:/config
      - /media/usb_4tb:/media/usb_4tb
      - /media/usb_8tb:/media/usb_8tb
      - /media/sata_2tb/Downloads:/downloads
    ports:
      - 6767:6767
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

  readarr:
    image: ghcr.io/linuxserver/readarr:develop
    restart: unless-stopped
    volumes:
      - ./readarr-config:/config
      - /media/sata_2tb/Books:/books
      - /media/sata_2tb/Downloads:/downloads
    ports:
      - 8787:8787
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

  prowlarr:
    image: ghcr.io/linuxserver/prowlarr:latest
    restart: unless-stopped
    volumes:
      - ./prowlarr-config:/config
    ports:
      - 9696:9696
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Guayaquil

  openbooks:
    image: evanbuss/openbooks:latest
    restart: unless-stopped
    volumes:
      - /media/sata_2tb/Books/OpenBooks:/books
    ports:
      - 12450:80
    command: --name <USER_HERE> --persist
    environment:
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
sudo ufw allow 3579/tcp
sudo ufw allow 8989/tcp
sudo ufw allow 7878/tcp
sudo ufw allow 8686/tcp
sudo ufw allow 6595/tcp
sudo ufw allow 6767/tcp
sudo ufw allow 8787/tcp
sudo ufw allow 9696/tcp
sudo ufw allow 12450/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
