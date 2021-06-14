# Fleet

[Fleet](https://github.com/linuxserver/fleet) is a web UI that displays maintained Docker images from our own repositories, it serves a central place to display all your docker images. This service has an official image on [Docker Hub](https://hub.docker.com/r/linuxserver/fleet) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/docker/fleet
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  fleet:
    image: ghcr.io/linuxserver/fleet:latest
    restart: unless-stopped
    volumes:
      - ./config:/config
    ports:
      - 8080:8080
    depends_on:
      - db
    environment:
      - PUID=1000
      - PGID=1000
      - fleet_admin_authentication_type=DATABASE
      - fleet_database_url=jdbc:mariadb://db/fleet
      - fleet_database_username=fleet
      - fleet_database_password=CHANGE_THIS
      - TZ=America/Guayaquil

  db:
    image: mariadb:10
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=CHANGE_THIS
      - MYSQL_DATABASE=fleet
      - MYSQL_USER=fleet
      - MYSQL_PASSWORD=CHANGE_THIS
```

!!! note
    In the case of the `PUID` and `PGID` environment variables, `1000` corresponds to the user's UID and GID respectively. You can find the values for your own user by running `id $whoami`.

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 8080/tcp
```

The default user and password are: `admin`:`admin`, you should create a new user and remove this initial admin user.

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
