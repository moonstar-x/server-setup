# Homepage

[Homepage](https://github.com/benphelps/homepage) is a homepage for your server, it allows to have links to your self hosted services. It also has some nice integrations with some of these services so it can display information about them from this homepage.

There is an official image for this service that we'll use: [ghcr.io/gethomepage/homepage](https://github.com/benphelps/homepage/pkgs/container/homepage).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/management/homepage
```

## Docker Compose

*Homepage* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/gethomepage/homepage:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - homepage
    volumes:
      - ./config:/app/config
      - ./images:/app/public/images
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /media/sata_2tb:/media/sata_2tb:ro
      - /media/usb_4tb:/media/usb_4tb:ro
      - /media/usb_4tb_2:/media/usb_4tb_2:ro
      - /media/usb_8tb:/media/usb_8tb:ro
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.homepage.rule: Host(`home.arpa`)
      traefik.http.routers.homepage.entrypoints: local
      traefik.http.routers.homepage.service: homepage@docker
      traefik.http.services.homepage.loadbalancer.server.port: 3000

networks:
  proxy_external:
    external: true
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
