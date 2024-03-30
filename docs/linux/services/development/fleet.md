# Fleet

[Fleet](https://github.com/linuxserver/fleet) is a web UI that displays maintained Docker images from our own repositories, it serves a central place to display all your docker images.

There is an official image for this service that we'll use: [ghcr.io/linuxserver/fleet](https://hub.docker.com/r/linuxserver/fleet).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/development/fleet
```

## Docker Compose

*Fleet* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  fleet:
    image: ghcr.io/linuxserver/fleet:latest
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - 30000:8080
    volumes:
      - ./config:/config
    environment:
      TZ: America/Guayaquil
      PUID: 1000
      PGID: 1000
      fleet_admin_authentication_type: DATABASE
      fleet_database_url: jdbc:mariadb://db/fleet
      fleet_database_username: fleet
      fleet_database_password: DATABASE_PASSWORD

  db:
    image: mariadb:10
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/mysql
    environment:
      TZ: America/Guayaquil
      MYSQL_ROOT_PASSWORD: DATABASE_PASSWORD
      MYSQL_DATABASE: fleet
      MYSQL_USER: fleet
      MYSQL_PASSWORD: DATABASE_PASSWORD
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    Make sure to change `DATABASE_PASSWORD` to a custom secret value.

## Post-Installation

The default user and password are: `admin`:`admin`, you should create a new user and remove this initial admin user.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
