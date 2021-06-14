# Webframes

[Webframes](https://github.com/moonstar-x/webframes) is a little webapp that frames other pages. This will be useful to have a central place where you can access all the aforementioned web services. This service has an image on [Docker Hub](https://hub.docker.com/r/moonstarx/webframes) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/other/webframes
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  internal:
    image: moonstarx/webframes:latest
    restart: unless-stopped
    ports:
      - 80:4000
    volumes:
      - ./internal-data:/opt/app/backend/data
    environment:
      - TZ=America/Guayaquil

  external:
    image: moonstarx/webframes:latest
    restart: unless-stopped
    ports:
      - 4000:4000
    volumes:
      - ./external-data:/opt/app/backend/data
    environment:
      - TZ=America/Guayaquil
```

This will create two different instances of this service, one that should be used for local usage (with the private IP) and the other one that will be accessible from outside.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 80/tcp
sudo ufw allow 4000/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
