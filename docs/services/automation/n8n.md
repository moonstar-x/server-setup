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
    ports:
      - 5678:5678
    environment:
      - TZ=America/Guayaquil
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=CHANGE_THIS
      - N8N_BASIC_AUTH_PASSWORD=CHANGE_THIS
```

!!! note
    Make sure to change `CHANGE_THIS` to a custom value.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 5678/tcp
```

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
