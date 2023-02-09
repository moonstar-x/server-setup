# Ghost

!!! warning "Deprecation Warning"
    This service was never really used, though I feel like I may give this a more serious try in the future. Currently, I have opted to use [Medium](https://medium.com/@moonstar-x), at least for now.

[Ghost](https://ghost.org/) is a self-hosted blogging platform. This service has an official image available on [Docker Hub](https://hub.docker.com/r/_/ghost) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/ghost
```

## Docker Compose

*Ghost* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  ghost:
    image: ghost:latest
    restart: unless-stopped
    depends_on:
      - db
    ports:
      - 15800:2368
    volumes:
      - ./content:/var/lib/ghost/content
    environment:
      - TZ=America/Guayaquil
      - database__client=mysql
      - database__connection__host=db
      - database__connection__user=root
      - database__connection__password=CHANGE_THIS
      - database__connection__database=ghost
      - url=https://example.com

  db:
    image: mysql:8.0
    restart: unless-stopped
    volumes:
      - ./data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=CHANGE_THIS
```

!!! note
    Make sure to replace `CHANGE_THIS` with something else. Also, the `url` variable should be set to the URL of your blog.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 15800/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
