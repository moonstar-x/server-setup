# Valheim

[Valheim](https://store.steampowered.com/app/892970/Valheim/) is an exploration game set in a Viking mythology world. This game server has an un-official image available on [Docker Hub](https://hub.docker.com/r/lloesche/valheim-server) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the game server's data will be saved.

```bash
mkdir ~/games/valheim
```

## Docker Compose

The game server will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  valheim:
    image: lloesche/valheim-server:latest
    cap_add:
      - SYS_NICE
    ports:
      - 2456:2456/udp
      - 2457:2457/udp
    volumes:
      - ./config:/config
      - ./data:/opt/valheim
    environment:
      - TZ=America/Guayaquil
      - SERVER_NAME=CHANGE_THIS
      - WORLD_NAME=world
      - SERVER_PASS=CHANGE_THIS
      - SERVER_PUBLIC=false
      - PUID=1000
      - PGID=1000
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 2456/udp
sudo ufw allow 2457/udp
```

## Starting for the First Time

Start up the game server with:

```bash
docker-compose up -d
```

## Starting/Stopping the Server

Since the server is not supposed to be restarted on server boot, you should manually start the server with:

```bash
docker-compose start
```

And stop it with:

```bash
docker-compose stop
```
