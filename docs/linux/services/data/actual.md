# Actual

[Actual](https://actualbudget.org/) is a self-hosted budgeting application.

There is an official image for this service that we'll use: [actualbudget/actual-server](https://hub.docker.com/r/actualbudget/actual-server/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/data/actual
```

## Docker Compose

*Actual* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  actual:
    image: actualbudget/actual-server:latest
    restart: unless-stopped
    ports:
      - 40000:5006
    volumes:
      - ./data:/data
    environment:
      TZ: America/Guayaquil
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
