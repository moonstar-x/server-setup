# Scrutiny

[Scrutiny](https://github.com/AnalogJ/scrutiny) is a S.M.A.R.T monitoring tool that uses `smartd`.

There is an official image for this service that we'll use: [ghcr.io/analogj/scrutiny](https://github.com/AnalogJ/scrutiny).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/observability/scrutiny
```

## Docker Compose

*Scrutiny* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: ghcr.io/analogj/scrutiny:master-web
    restart: unless-stopped
    networks:
      default:
      proxy_external:
        aliases:
          - scrutiny
    depends_on:
      influx:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/api/health"]
      interval: 5s
      timeout: 10s
      retries: 20
      start_period: 10s
    volumes:
      - ./config:/config
    environment:
      TZ: America/Guayaquil
      SCRUTINY_WEB_INFLUXDB_HOST: influx
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.scrutiny.rule: Host(`scrutiny.alpha.example.com`) || Host(`scrutiny.alpha.home.example.com`)
      traefik.http.routers.scrutiny.entrypoints: local-https
      traefik.http.routers.scrutiny.tls: true
      traefik.http.routers.scrutiny.tls.certresolver: le
      traefik.http.routers.scrutiny.service: scrutiny@docker
      traefik.http.services.scrutiny.loadbalancer.server.port: 8080

  collector:
    image: ghcr.io/analogj/scrutiny:master-collector
    restart: unless-stopped
    cap_add:
      - SYS_RAWIO
    devices:
      - /dev/sda:/dev/sda
      - /dev/sdb:/dev/sdb
      - /dev/sdc:/dev/sdc
      - /dev/sdd:/dev/sdd
      - /dev/sde:/dev/sde
    depends_on:
      web:
        condition: service_healthy
    volumes:
      - /run/udev:/run/udev:ro
    environment:
      TZ: America/Guayaquil
      COLLECTOR_API_ENDPOINT: http://web:8080
      COLLECTOR_HOST_ID: scrutiny-collector

  influx:
    image: influxdb:2.2
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8086/health"]
      interval: 5s
      timeout: 10s
      retries: 20
    volumes:
      - ./data:/var/lib/influxdb2
    environment:
      TZ: America/Guayaquil

networks:
  proxy_external:
    external: true
```

!!! note
    Under `devices` make sure you're passing your hard drives. You can check `blkid` to see which device blocks to pass.

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

## Updating Results

To manually update the results, run the following from inside the container's data folder:

```bash
docker compose run --rm collector scrutiny-collector-metrics run
```
