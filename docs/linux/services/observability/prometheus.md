# Prometheus

[Prometheus](https://prometheus.io/) is a time series database for monitoring systems, this will serve as the backend of the monitoring service.

There is an official image for this service that we'll use: [prom/prometheus](https://hub.docker.com/r/prom/prometheus).

## Pre-Installation

We'll create a folder in the main user's home where all the service's data will be saved.

```bash
mkdir ~/services/observability/prometheus
```

Inside this folder, create a `data` folder:

```bash
mkdir data
```

And also, create a `prometheus.yml` file:

```bash
nano prometheus.yml
```

Which should have the following:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    scrape_interval: 15s
    static_configs:
    - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
    - targets: ["host.docker.internal:9100"]
```

### External Network

Since this service needs to interoperate with another one, we'll need to have them inside the same network. Make sure to have created the `grafana_external` network before defining the `docker-compose.yml` file. If you haven't created this network, you can do so with:

```bash
docker network create grafana_external
```

## Docker Compose

*Prometheus* will be run using *Docker Compose*. The content of the `docker-compose.yml` file is as follows:

```yaml
services:
  web:
    image: prom/prometheus:latest
    restart: unless-stopped
    user: 1000:1000
    extra_hosts:
      - host.docker.internal:host-gateway
    networks:
      default:
      grafana_external:
      proxy_external:
        aliases:
          - prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./data:/prometheus
    environment:
      TZ: America/Guayaquil
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.prometheus.rule: Host(`prometheus.home.example.com`, `prometheus.vpn.example.com`)
      traefik.http.routers.prometheus.entrypoints: local-https
      traefik.http.routers.prometheus.tls: true
      traefik.http.routers.prometheus.tls.certresolver: le
      traefik.http.routers.prometheus.service: prometheus@docker
      traefik.http.services.prometheus.loadbalancer.server.port: 9090

  exporter:
    image: prom/node-exporter:latest
    restart: unless-stopped
    user: 1000:1000
    network_mode: host
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^(sys|proc|dev|host|etc)($$|/)'
    environment:
      TZ: America/Guayaquil

networks:
  grafana_external:
    external: true
  proxy_external:
    external: true
```

!!! note
    The `exporter` service needs to have `network_mode: host` in order to have access to the main network interface to actually have valid network usage data.

### Reverse Proxy

This service is exposed by a reverse proxy. More specifically, it is using [Traefik](../networking/traefik.md).

For this reason, you will see that this service has:

1. A directive to connect it to the `proxy_external` external network.
2. A container alias for the `proxy_external` network.
3. A number of labels with names starting with `traefik`.

If you're not using a reverse proxy, feel free to remove these from the `docker-compose.yml` file.
Keep in mind you might need to bind the ports to connect to the service instead.

## Post-Installation

We'll need to allow the service's port on our firewall.

```bash
sudo ufw allow 9100/tcp
```

## Running

Start up the service with:

```bash
docker compose up -d
```

That's it! The service will auto-start on system startup and restart on failure.
