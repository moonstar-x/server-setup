# Rustdesk

[Rustdesk](https://rustdesk.com) is an open source remote desktop software that allows us to self-host an ID and relay server to act as a middleman to connect two computers remotely.

There is an official image for this service that we'll use: [rustdesk/rustdesk-server](https://hub.docker.com/r/rustdesk/rustdesk-server).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/management/rustdesk
```

## Docker Compose

*Rustdesk* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  server:
    image: rustdesk/rustdesk-server:latest
    restart: unless-stopped
    command: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    volumes:
      - ./data:/root
    environment:
      TZ: America/Guayaquil

  relay:
    image: rustdesk/rustdesk-server:latest
    restart: unless-stopped
    command: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    volumes:
      - ./data:/root
    environment:
      TZ: America/Guayaquil
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.

## Post Installation

Inside the `data` folder you'll find an `id_ed25519.pub` file. This is your public key. Every client that wants to connect to this ID and relay server must set this value.
