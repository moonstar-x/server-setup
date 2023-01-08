# Portainer (Agent)

[Portainer](https://www.portainer.io/) is a web UI for Docker which allows us to have an insight on all the containers running on our server.
An agent is a service that can expose a particular Docker server through Portainer hosted on another machine.

In this case we'll use this to manage the monitor's Docker service.

This service has an official image on [Docker Hub](https://hub.docker.com/r/portainer/agent) which we'll use.

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/portainer-agent
```

Now, head over to the [Portainer](../../services/docker/portainer.md) web dashboard and access:

```text
Settings > Environments > Add environment > Docker Standalone > Edge Client
```

Set a name for it and make sure that the Portainer URL is accessible by the agent, then click *Create*.

You will get a preview of a `docker` command with two environment variables: `EDGE_ID` and `EDGE_KEY`. Save these
values because we'll need them later.

## Docker Compose

The service will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
version: "3.9"

services:
  agent:
    image: portainer/agent:latest
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
      - /:/host
      - ./data:/data
    environment:
      - TZ=America/Guayaquil
      - EDGE=1
      - EDGE_ID=CHANGE_ME
      - EDGE_KEY=CHANGE_ME
      - EDGE_INSECURE_POLL=1
```

!!! note
    Replace `CHANGE_ME` with the appropriate variables taken before.

## Running

Start up the service with:

```bash
docker-compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
