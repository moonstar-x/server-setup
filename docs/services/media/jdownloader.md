# JDownloader

[JDownloader](https://jdownloader.org/) is a download client that makes downloading from direct links a breeze. There is no official image for *JDownloader*, however we'll use one available on [Docker Hub](https://hub.docker.com/r/jlesage/jdownloader-2).

## Pre-Installation

We'll create a folder in the main user's home where all the download client's data will be saved.

```bash
mkdir ~/media/jdownloader
```

## Docker Compose

*JDownloader* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  jdownloader:
    image: jlesage/jdownloader-2:latest
    restart: unless-stopped
    volumes:
      - ./config:/config
      - /media/sata_2tb/Downloads:/output
    ports:
      - 3129:3129
      - 5800:5800
    environment:
      - TZ=America/Guayaquil
      - USER_ID=1000
      - GROUP_ID=1000
```

!!! note
    In the case of the `USER_ID` and `GROUP_ID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 3129/tcp
sudo ufw allow 5800/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
