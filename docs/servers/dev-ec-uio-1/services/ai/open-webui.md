# Open WebUI

[Open WebUI](https://docs.openwebui.com) is a self-hosted frontend to interact with LLMs.

There is an official image for this service that we'll use: [ghcr.io/open-webui/open-webui](https://github.com/open-webui/open-webui).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/ai/open-webui
```

## Docker Compose

*Open WebUI* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/open-webui/open-webui:main
    restart: unless-stopped
    extra_hosts:
      - host.docker.internal:host-gateway
    networks:
      default:
      proxy_external:
        aliases:
          - open-webui
    volumes:
      - ./data:/app/backend/data
    environment:
      TZ: America/Guayaquil
      OLLAMA_BASE_URL: ${OLLAMA_BASE_URL}
      WEBUI_SECRET_KEY: ${SECRET_KEY}
      WEBUI_URL: https://${DOMAIN_OPEN_WEBUI}
      ENABLE_SIGNUP: false
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.open-webui.rule: Host(`${DOMAIN_OPEN_WEBUI}`)
      traefik.http.routers.open-webui.entrypoints: public
      traefik.http.routers.open-webui.service: open-webui@docker
      traefik.http.services.open-webui.loadbalancer.server.port: 8080
      
  proxy:
    image: nginx:alpine
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    environment:
      TZ: America/Guayaquil

networks:
  proxy_external:
    external: true
```

!!! note
    The `proxy` container is completely optional, you might not need it. I'm using it to access an Ollama instance on a separate machine on my local network without having to expose the `web` container itself.

### `nginx.conf`

The contents of this file are as follows:

```text
events {}

http {
    server {
        listen 61000;

        location / {
            proxy_pass http://192.168.100.43:11434;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

!!! info
    This proxy configuration acts as a route to forward all requests to the host machine on port `61000` to a machine on my local network on `192.168.100.43:11434` which is an Ollama instance on a separate machine.

    This way, from within the `web` container this LAN service becomes accessible on `host.docker.internal:61000`.

### Secrets

Make sure to create a `.env` file with the following structure:

```text
DOMAIN_OPEN_WEBUI=
OLLAMA_BASE_URL=
SECRET_KEY=
```

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Post-Installation

In case you decide to use the `proxy` container as how I'm using it, you'll need to create a firewall rule.

```bash
sudo ufw allow 61000/tcp
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
