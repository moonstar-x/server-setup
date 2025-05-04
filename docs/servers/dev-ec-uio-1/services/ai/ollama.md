# Ollama

[Ollama](https://ollama.com) is a self-hosted tool that allows you to run open-source models on your own hardware.

There is an official image for this service that we'll use: [ollama/ollama](https://hub.docker.com/r/ollama/ollama/).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/ai/ollama
```

## Docker Compose

*Ollama* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ollama/ollama:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - ollama
    volumes:
      - ./ollama:/root/.ollama
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.ollama.rule: Host(`${DOMAIN_OLLAMA_LOCAL}`)
      traefik.http.routers.ollama.entrypoints: local-https
      traefik.http.routers.ollama.tls: true
      traefik.http.routers.ollama.tls.certresolver: le
      traefik.http.routers.ollama.service: ollama@docker
      traefik.http.services.ollama.loadbalancer.server.port: 11434

networks:
  proxy_external:
    external: true
```

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_OLLAMA_LOCAL=
```

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Installing Models

Inside the service folder, run the following command:

```bash
docker compose exec web /bin/sh
```

You can then pull your favorite models like so:

```bash
ollama run qwen2.5:3b
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
