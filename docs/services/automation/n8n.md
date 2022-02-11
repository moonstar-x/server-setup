# n8n

[n8n](https://n8n.io/) is a self-hosted node based automation to run run jobs based on triggers similarly to IFTTT. There is an official image for *n8n* on [Docker Hub](https://hub.docker.com/r/n8nio/n8n) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/automation/n8n
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    volumes:
      - ./data:/home/node/.n8n
      - ./local-files:/home/node/local-files
    ports:
      - 5678:5678
    environment:
      - TZ=America/Guayaquil
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=CHANGE_THIS
      - N8N_BASIC_AUTH_PASSWORD=CHANGE_THIS
      - WEBHOOK_URL=https://example.com

  mongo:
    image: mongo:latest
    restart: unless-stopped
    ports:
      - 56787:27017
    volumes:
      - ./mongo-data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=CHANGE_THIS
      - MONGO_INITDB_ROOT_PASSWORD=CHANGE_THIS
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

The `local-files` volume for `n8n` will serve as a way to load binary files from the filesystem in our workflows.

We also need a database for workflows that require data persistence, in this case we've chosen `mongo` but any database that is supported by `n8n` can work just fine.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 5678/tcp
sudo ufw allow 56787/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.