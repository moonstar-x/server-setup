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
  web_home:
    image: ghcr.io/gethomepage/homepage:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - homepage-home
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
      HOMEPAGE_VAR_DOMAIN: home.example.com
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.homepage-home.rule: Host(`home.example.com`)
      traefik.http.routers.homepage-home.entrypoints: local-https
      traefik.http.routers.homepage-home.service: homepage-home@docker
      traefik.http.services.homepage-home.loadbalancer.server.port: 3000
      traefik.http.routers.homepage-home.tls: true
      traefik.http.routers.homepage-home.tls.certresolver: le

  web_vpn:
    image: ghcr.io/gethomepage/homepage:latest
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - homepage-vpn
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
      HOMEPAGE_VAR_DOMAIN: vpn.example.com
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.homepage-vpn.rule: Host(`vpn.example.com`)
      traefik.http.routers.homepage-vpn.entrypoints: local-https
      traefik.http.routers.homepage-vpn.service: homepage-vpn@docker
      traefik.http.services.homepage-vpn.loadbalancer.server.port: 3000
      traefik.http.routers.homepage-vpn.tls: true
      traefik.http.routers.homepage-vpn.tls.certresolver: le

networks:
  proxy_external:
    external: true
```

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
