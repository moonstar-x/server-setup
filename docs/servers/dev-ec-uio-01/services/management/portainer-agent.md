# Portainer Agent

[Portainer](https://www.portainer.io/) is a web UI for Docker which allows us to have an insight on all the containers running on our server.

The *Portainer* agent allows you to expose the machine's Docker management to another *Portainer* service hosted elsewhere. We're using the agent in this case to use
the main [Portainer](../../../bravo/services/management/portainer.md) service to manage this one.

There is an official image for this service that we'll use: [portainer/agent](https://hub.docker.com/r/portainer/agent).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/management/portainer-agent
```

## Docker Compose

*Portainer Agent* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  agent:
    image: portainer/agent:latest
    restart: unless-stopped
    ports:
      - 9001:9001
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    environment:
      TZ: America/Guayaquil
```

### Add to Portainer

In your main *Portainer* service,

1. Head over to `Environments > Add environment`.
2. Pick `Docker Standalone` and click on `Start wizard`.
3. Select `Agent`, set a `Name` for your agent and paste your server's address with `9001` as the port into `Environment address`.
4. Finally, click `Connect`.

And that's it, you can now manage your server's Docker services from your main *Portainer* service.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
