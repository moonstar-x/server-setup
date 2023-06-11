# Custom Automation Service

This service is completely custom made for my needs so chances are this will not be of any
use for you.

The only reason I went out of my way to create my own service was because I was finding myself using way too much the `Code` node in [n8n](https://n8n.io/). Given this, maintaining my workflows wasn't really that easy and at this point it would be a lot better
for me to just maintain an actual code based workflow system rather than the workflows I had there.

This service has an Docker image hosted on [GitHub Packages](https://github.com/moonstar-x/automation-service/pkgs/container/automation-service) which will be used.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/automation/automation-service
```

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  service:
    image: ghcr.io/moonstar-x/automation-service:latest
    restart: unless-stopped
    user: 1000:1000
    volumes:
      - ./data:/opt/app/data
      - ./config:/opt/app/config
    ports:
      - 5678:5678
    environment:
      - TZ=America/Guayaquil
```

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
