# Actual

[Actual](https://actualbudget.org/) is a self-hosted budgeting application. This service has an official image on [Docker Hub](https://hub.docker.com/r/actualbudget/actual-server/) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/data/actual
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  actual:
    image: actualbudget/actual-server:latest
    restart: unless-stopped
    volumes:
      - ./data:/data
    ports:
      - 5006:5006
    environment:
      - TZ=America/Guayaquil
```

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 5006/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
