# FreshRSS

[FreshRSS](https://www.freshrss.org/) is an RSS feed reader. For this service we'll use an image from **LinuxServer** available on [Docker Hub](https://hub.docker.com/r/linuxserver/freshrss) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the media server's data will be saved.

```bash
mkdir ~/media/freshrss
```

## Docker Compose

*FreshRSS* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  freshrss:
    image: lscr.io/linuxserver/freshrss:latest
    restart: unless-stopped
    ports:
      - 5200:80
    volumes:
      - ./data:/config
    environment:
      - TZ=America/Guayaquil
      - PUID=1000
      - PGID=1000
```

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 5200/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
