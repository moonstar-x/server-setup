# TeamSpeak 3

[TeamSpeak 3](https://www.teamspeak.com/en/) is a gaming focused voice server.

There is an official image for this service that we'll use: [teamspeak](https://hub.docker.com/_/teamspeak).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/games/teamspeak
```

## Docker Compose

*TeamSpeak 3* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  server:
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
      TZ: America/Guayaquil
      TS3SERVER_DB_PLUGIN: ts3db_mariadb
      TS3SERVER_DB_SQLCREATEPATH: create_mariadb
      TS3SERVER_DB_HOST: db
      TS3SERVER_DB_USER: root
      TS3SERVER_DB_NAME: ${MYSQL_DATABASE}
      TS3SERVER_DB_PASSWORD: ${MYSQL_PASSWORD}
      TS3SERVER_DB_WAITUNTILREADY: 30
      TS3SERVER_LICENSE: accept

  db:
    image: mariadb:10
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/mysql
    environment:
      TZ: America/Guayaquil
      MYSQL_ROOT_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
MYSQL_DATABASE=
MYSQL_PASSWORD=
```

## Getting Server Auth Tokens

After the container has been created, check its logs and save the `serveradmin` login details. This is very important in case you get locked out of your server or if you need to change some settings through ServerQuery.

Use:

```bash
docker compose logs server -f
```

You should also find here the privilege key to set up your user as the server administrator.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
