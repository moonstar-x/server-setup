# Minecraft (Paper)

[Minecraft (Paper)](https://papermc.io/) is a plugin based server for [Minecraft](https://minecraft.net). This game server has an un-official image available on [Docker Hub](https://hub.docker.com/r/itzg/minecraft-server) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the game server's data will be saved.

```bash
mkdir ~/games/minecraft-paper
```

## Docker Compose

The game server will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  minecraft:
    image: itzg/minecraft-server:latest
    ports:
      - 25565:25565
    volumes:
      - ./data:/data
    environment:
      - TZ=America/Guayaquil
      - EULA=TRUE
      - MEMORY=2G
      - VERSION=LATEST
      - TYPE=PAPER
```

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 25565/tcp
```

## Configuration

Inside the `data` folder there will be all the server's information, this includes the `server.properties` file which includes all the configuration for the server. You may modify this.

## Adding Plugins

You may add plugins for PaperMC inside `data/plugins`, just make sure to restart the container when adding or removing plugins.

## Running Commands

You can run commands inside the console through RCON by running:

```bash
docker-compose exec minecraft rcon-cli <command>
```

For more information, check:

```bash
docker-compose exec minecraft rcon-cli --help
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
