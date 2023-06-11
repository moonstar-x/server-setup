# TeamSpeak 3

[TeamSpeak 3](https://www.teamspeak.com/en/) is a gaming focused voice server. This service has an official image available on [Docker Hub](https://hub.docker.com/_/teamspeak) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/games/teamspeak
```

## Docker Compose

The bot will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  teamspeak:
    image: teamspeak:latest
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - 9987:9987/udp
      - 10011:10011
      - 30033:30033
    volumes:
      - ./data:/var/ts3server
    environment:
      - TZ=America/Guayaquil
      - TS3SERVER_DB_PLUGIN=ts3db_mariadb
      - TS3SERVER_DB_SQLCREATEPATH=create_mariadb
      - TS3SERVER_DB_HOST=db
      - TS3SERVER_DB_USER=root
      - TS3SERVER_DB_NAME=teamspeak
      - TS3SERVER_DB_PASSWORD=CHANGE_THIS
      - TS3SERVER_DB_WAITUNTILREADY=30
      - TS3SERVER_LICENSE=accept

  db:
    image: mariadb:10
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=CHANGE_THIS
      - MYSQL_DATABASE=teamspeak
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Getting Server Auth Tokens

After the container has been created, check its logs and save the `serveradmin` login details. This is very important in case you get locked out of your server or if you need to change some settings through ServerQuery.

Use:

```bash
docker logs teamspeak_teamspeak_1 --follow
```

You should also find here the privilege key to set up your user as the server administrator.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 10011/tcp
sudo ufw allow 30033/tcp
sudo ufw allow 9987/udp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
